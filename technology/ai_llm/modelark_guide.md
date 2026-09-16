# ModelArk: ByteDance's Model-as-a-Service Platform, the 火山方舟 / ModelArk Naming Puzzle, and What a Regulated Buyer Can Actually Establish

> **Author:** Jack Liu Shurui  
> **Role:** Solution Architect, Cymbal Bank  
> **Date:** September 2026  
> **Version:** 1.0  
> **Repository:** github.com/jackliusr/research  

---

## Table of Contents

1. [The Overview and the Naming Resolution](#1-the-overview-and-the-naming-resolution)
2. [The Company and the Cloud Context](#2-the-company-and-the-cloud-context)
3. [The Model Catalogue](#3-the-model-catalogue)
4. [The Platform Capabilities](#4-the-platform-capabilities)
5. [The Developer Surface](#5-the-developer-surface)
6. [The Commercial Model and the Pricing](#6-the-commercial-model-and-the-pricing)
7. [The Compliance and Data-Governance Position](#7-the-compliance-and-data-governance-position)
8. [The Security Posture](#8-the-security-posture)
9. [The Scale and Performance Claims](#9-the-scale-and-performance-claims)
10. [The Competitive Position](#10-the-competitive-position)
11. [The Regulated-Institution Angle](#11-the-regulated-institution-angle)
12. [The Cymbal Bank Worked Example](#12-the-cymbal-bank-worked-example)
13. [The Claims Audit](#13-the-claims-audit)
14. [What Could Not Be Verified](#14-what-could-not-be-verified)
15. [Glossary](#15-glossary)
16. [Cross-References and Further Reading](#16-cross-references-and-further-reading)
17. [Closing Summary](#17-closing-summary)

---

## 1. The Overview and the Naming Resolution

This section exists because the naming is the hard part. The capability is easy to summarise; the vocabulary around it is genuinely confusing, and getting it wrong means a bank's architect signs up for the wrong product in the wrong jurisdiction under the wrong legal entity. Read this section before anything else.

### 1.1 The one-paragraph summary

**火山方舟 (VolcanoArk)** is the model-as-a-service (MaaS) platform operated inside **火山引擎 (Volcano Engine, usually written "Volcengine")**, ByteDance's cloud computing business. It provides inference, fine-tuning/training, evaluation, a knowledge base for retrieval, a plugin system, batch jobs, prompt tooling and an agent runtime over a catalogue of large models — ByteDance's own **Doubao / Seed** family first among them. The same platform is sold outside mainland China under a different product name, **ModelArk**, by **BytePlus**, ByteDance's international enterprise technology arm. Same product lineage, same console metaphor, different brand, different regions, different legal entity, different contract, and a partly different model catalogue.

### 1.2 The mapping table — what to type and where

| Layer | Mainland-China name | International name | What it actually is | Where you sign up |
|---|---|---|---|---|
| Cloud | 火山引擎 / Volcengine | BytePlus | The hyperscale cloud (compute, storage, CDN, DB, ML) | console.volcengine.com / console.byteplus.com |
| Model-as-a-service platform | 火山方舟 / VolcanoArk | **ModelArk** | Inference, fine-tune, eval, RAG KB, plugins, agents | volcengine.com/product/ark → console / ai.byteplus.com/ark |
| Model family (foundation LLMs) | 豆包大模型 / Doubao | **Dola Seed** (branded); also "Seed" | The models you call: text, vision, reasoning, embeddings | via the platform above |
| Media models | 即梦/Seedream, Seedance | Dreamina Seedream, Dreamina Seedance | Image and video generation models | via the platform above |
| Consumer AI app | 豆包 (Doubao app) | Cici / Dola (consumer apps differ by market) | The ChatGPT-equivalent end-user product — **NOT** the platform | app stores |
| Agent/app dev platform | 扣子 / Coze | Coze | No-code and low-code AI app/agent builder | coze.cn / coze.com |
| Model research | ByteDance Seed (字节跳动 Seed) | ByteDance Seed | The research team publishing the model family | seed.bytedance.com |

✅ **Verified (vendor documentation + product pages, accessed 2026-09-16):** the mainland platform is documented at [volcengine.com/product/ark](https://www.volcengine.com/product/ark) and [docs.volcengine.com/docs/82379](https://docs.volcengine.com/docs/82379), and its console lives at `ark.volcengine.com/region:cn-beijing/...`. The international product is documented at [docs.byteplus.com/en/docs/ModelArk](https://docs.byteplus.com/en/docs/ModelArk/1099455) with the product page at [byteplus.com/en/product/modelark](https://www.byteplus.com/en/product/modelark) and the console at `ai.byteplus.com/ark`.

### 1.3 The naming-gate finding, stated precisely

The candidate identity supplied to this guide was **partly right and partly inverted**. The corrections that matter:

1. **"ModelArk" is not the mainland-China product name.** In mainland China the platform is branded **火山方舟 / VolcanoArk**. The English string "ModelArk" is the name used by **BytePlus** on **docs.byteplus.com**, `byteplus.com/en/product/modelark` and `ai.byteplus.com/ark` — i.e. the **international** variant. The international documentation's own navigation, page titles and breadcrumbs all read "ModelArk".
2. **"Volcano Ark" is a transliteration of 方舟, not a separate product.** 方舟 means "ark" (as in Noah's ark). Baidu Baike gives the product's foreign name as **VolcanoArk** — [baike.baidu.com/item/火山方舟](https://baike.baidu.com/item/%E7%81%AB%E5%B1%B1%E6%96%B9%E8%88%9F/63145443) (accessed 2026-09-16). Treat "Volcano Ark", "VolcanoArk", "火山方舟" and "Ark" (in `ark.volcengine.com`) as one product.
3. **The consumer brand and the platform brand share a word and nothing else.** 豆包 / Doubao is a consumer AI assistant app **and** the name historically used for ByteDance's model family on the mainland platform. It is not the platform. The platform is 火山方舟.
4. **The international model family is re-branded.** On BytePlus the flagship model family is marketed as **"Dola Seed 2.0"** with model IDs such as `dola-seed-2-1-turbo` and `seed-2-0-lite-260228`, whereas the same lineage appears on the mainland side under **豆包 / Doubao** and on the research site as **ByteDance Seed** ([ai.byteplus.com/en/model](https://ai.byteplus.com/en/model), [seed.bytedance.com/en/models](https://seed.bytedance.com/en/models), accessed 2026-09-16). A mainland-China build and an international build of "the same product" will therefore not have the same model identifiers.
5. **BytePlus is the legal counterparty for the international variant.** BytePlus pages are footed "© 2026 BytePlus Pte Ltd." — a Singapore entity — and its trust centre documents alignment with **Singapore PDPA** and **EU GDPR** ([byteplus.com/en/trust-center](https://www.byteplus.com/en/trust-center), accessed 2026-09-16). The mainland variant operates under Chinese law and the CAC generative-AI filing regime (see §7). **These are two different regulatory universes behind one brand.**

❓ **Unverified / reported:** whether the mainland product was ever *renamed* from an earlier "ModelArk"-style English label, or whether "ModelArk" was coined for the international market at launch. No primary renaming announcement was found. What *is* verified is that the same product carries two distinct product names today.

### 1.4 Disambiguation — things this guide is NOT about

| Similarly named thing | What it is | Relationship |
|---|---|---|
| **Volcano** (volcano.sh) | The CNCF **batch scheduler** for Kubernetes | ❌ Unrelated. Same English word, no connection to Volcengine or ByteDance. If a search result mentions `volcano.sh`, `vcctl`, or gang scheduling, it is not this product. |
| **火山** in other Chinese products | "Volcano" is a common Chinese cloud naming element (火山引擎 itself means "Volcano Engine") | ❌ Do not conflate the platform (火山方舟) with the cloud (火山引擎). |
| Generic "Ark" products | "Ark" is used by several unrelated vendors for unrelated products | ❌ The only Ark that matters here is reached at `ark.volcengine.com` (mainland) or `ai.byteplus.com/ark` (international). |
| 方舟 as a generic term | Chinese for "ark"; also used by other Chinese products and by game/app names | ❌ Always require the 火山 (Volcano) qualifier. |
| Coze / 扣子 | ByteDance's agent/app development platform | ✅ Genuinely related (same parent, integrated), but a *different product*. See §4.5. |

### 1.5 The API endpoint, verified

The supplied claim was that the endpoint is region-specific, e.g. `ark.cn-beijing.volces.com`. **This is correct for the mainland variant** and is the region-scoped host pattern the platform uses. The international variant's base URLs are documented explicitly ([docs.byteplus.com/en/docs/ModelArk/2191806](https://docs.byteplus.com/en/docs/ModelArk/2191806), accessed 2026-09-16):

- AP region (Johor, Malaysia), region ID `ap-southeast-1`: `https://ark.ap-southeast.bytepluses.com/api/v3`
- EU region (Dublin, Ireland), region ID `eu-west-1`: `https://ark.eu-west.bytepluses.com/api/v3`

⚠️ **Practical warning:** an inference endpoint created in one region **must** be called with that region's base URL, and the vendor states that API keys and model-activation status are themselves **region-isolated**. This is not cosmetic — it is a data-residency lever, and it is also a migration hazard. See §7.4.

### 1.6 What follows

Sections 2–6 are the factual product picture (company, catalogue, capabilities, developer surface, pricing). Sections 7–9 are the governance, security and performance claims, with source quality labelled. Section 10 positions the platform against its peers; §11 and §12 are the regulated-buyer assessment and the fictional Cymbal Bank worked example. §13–§14 are the claims audit and the honest list of what could not be established. A reader in a hurry should read §1, §7 and §13.

---

## 2. The Company and the Cloud Context

### 2.1 The corporate parent

**ByteDance** (字节跳动) is the Chinese technology company best known for TikTok and its mainland predecessor Douyin's content ecosystem. **火山引擎 (Volcano Engine / Volcengine)** is ByteDance's enterprise cloud business — the counterpart to AWS or Alibaba Cloud, built out of ByteDance's internal infrastructure and offered commercially. ✅ **Verified:** Volcengine operates a full console and documentation estate (`console.volcengine.com`, `docs.volcengine.com`) and the 火山方舟 product page is hosted under `volcengine.com/product/ark` (accessed 2026-09-16).

**BytePlus** is ByteDance's international enterprise technology arm, positioned on its own pages as "the AI-native enterprise partner from ByteDance", with the ByteDance logo in its footer and a Singapore legal entity, **BytePlus Pte Ltd.** ✅ **Verified** from [byteplus.com/en/product/modelark](https://www.byteplus.com/en/product/modelark) and [byteplus.com/en/trust-center](https://www.byteplus.com/en/trust-center) (accessed 2026-09-16).

⚠️ **Careful with this framing:** "BytePlus is the international arm of Volcengine" is the common shorthand and is broadly how the products relate, but the two estates are separately branded, separately documented, separately contracted and separately regioned. For procurement purposes treat **BytePlus and Volcengine as two vendors**, not one.

### 2.2 Launch and announcement timeline

| Date | Event | Source quality |
|---|---|---|
| **2023-06-28** | 火山方舟 (VolcanoArk) announced at the 火山引擎 **V-Tech 体验创新科技峰会** as a platform service (MaaS) for model fine-tuning, evaluation and inference, launched in **invite-only beta (邀测)** integrating models from Baichuan Intelligence, 出门问问, Fudan MOSS, IDEA Research Institute, 澜舟科技, MiniMax and Zhipu AI | ❓ Reported by [Baidu Baike](https://baike.baidu.com/item/%E7%81%AB%E5%B1%B1%E6%96%B9%E8%88%9F/63145443) citing 界面新闻 2023-06-28 (secondary; primary V-Tech materials not retrieved) |
| **2024-05-15** | 火山引擎 **FORCE 原动力大会**: ByteDance's self-developed **豆包大模型 (Doubao)** family formally opened to external customers, **9 members**, positioned as "厘时代" (per-milli, i.e. sub-cent) pricing. 火山引擎总裁 谭待 (Tan Dai) presented | ❓ Reported by [知乎 write-up of the FORCE conference](https://zhuanlan.zhihu.com/p/698265798) and Chinese business press (secondary) |
| **2024-06** | 火山方舟 opened to **individual developers**, offering the Doubao family "and a dozen-odd other models" | ❓ Secondary (Baike citing 百家号 2024-06-12) |
| **2025-06-11/12** | FORCE conference: **Doubao 1.6** and **Seedance 1.0 pro** released; new price cuts (see §6) | ✅ Reported in detail by [36Kr reprinting 界面新闻](https://m.36kr.com/p/3333345351248136) (2025-06-12) — third-party business press |
| **2025-07-30** | SeedEdit 3.0 image-editing model lands on 火山方舟 | ❓ Secondary (界面新闻 via Baike) |
| **2025-08-22** | **DeepSeek-V3.1** goes live on 火山方舟 | ❓ Secondary (IT之家 via Baike) |
| **2025-11-11** | **Doubao-Seed-Code** (豆包编程模型) released, fully opens API on 火山方舟; vendor claims 62.7% lower total cost of use than the industry average and "lowest price in China"; **TRAE** China edition integrates it | ❓ Secondary (百家号 via Baike); the 62.7% figure is a **vendor claim** |
| **2025-12-18** | **豆包助手 API** lands on 火山方舟: 日常沟通 (daily chat), 深度沟通 (deep chat), 联网搜索 (web search), 边想边搜 (search-while-thinking) endpoints | ❓ Secondary (百家号 via Baike) |
| **2026-03-24** | BytePlus publishes the **Dola Seed 2.0 Pro** announcement (Pro / Lite / Mini family) | ✅ [byteplus.com/en/blog/dola-seed-2-0-pro](https://www.byteplus.com/en/blog/dola-seed-2-0-pro) (accessed 2026-09-16) |
| **2026-08-21 / 2026-08-28** | BytePlus ModelArk publishes **price adjustments** on `deepseek-v4-flash-ga-260731` and on `deepseek-v4-flash-260425` / `deepseek-v4-pro-260425` | ✅ [docs.byteplus.com/en/docs/ModelArk/1544106](https://docs.byteplus.com/en/docs/ModelArk/1544106) (accessed 2026-09-16) |

❓ **Not verified:** the exact 2023 announcement wording, the full attendee/client list, and whether any English-language press release accompanied the 2023 launch. The 2023-06-28 date is consistent across secondary Chinese sources and is treated as **very likely correct but not primary-verified**.

### 2.3 The strategic logic, attributed

The strategic framing below is **reported** (Chinese business press) and is attributed rather than asserted:

- **A content company selling inference.** ByteDance built the model family to serve its own products first. Volcengine's product page states the models are validated across **50+ internal ByteDance business scenarios** with a daily volume in the **hundreds of billions of tokens** (⚠️ vendor claim, [volcengine.com/product/doubao-dy](https://www.volcengine.com/product/doubao-dy), accessed 2026-09-16). The commercial pitch is that your workload is powered by infrastructure already carrying ByteDance's own traffic.
- **Price as the market-entry weapon.** 界面新闻/36Kr report 谭待 framing the May 2024 cuts as clearing the cost obstacle to enterprise adoption, and the June 2025 cuts as targeting **agent-scale** consumption, on the argument that agents burn many more tokens per task and only become economical at a lower unit price ([36Kr/界面新闻, 2025-06-12](https://m.36kr.com/p/3333345351248136), accessed 2026-09-16).
- **Aggressive share ambitions.** The same piece reports that Volcengine set "very aggressive" targets and cites an **IDC** figure putting Doubao first in China's public-cloud LLM market at **46.4%** share. ⚠️ The IDC attribution is a **second-hand vendor-favourable statistic** — the underlying IDC report was not retrieved, so treat 46.4% as **reported, unverified**.
- **Where the platform sits relative to the consumer products.** The consumer 豆包 app, the Coze agent builder, the TRAE coding tool and 火山方舟 are one portfolio: the platform is the **supply layer** (models + inference + tools) under ByteDance's own and third parties' AI products. The consumer app is not the platform and is not sold to enterprises; the platform is. ✅ Position confirmed structurally from the product/navigation estates cited above.

### 2.4 Why this matters to a buyer

The company context has three procurement consequences:

1. **Single-vendor gravity.** The models, the inference platform, the agent runtime, the retrieval store, the coding tool and the consumer app are all ByteDance. Concentration risk is real and structural, not incidental — see §11.5.
2. **Different legal facts per market.** A mainland deployment engages Chinese law, Chinese filing obligations and a Chinese contracting entity; an international deployment engages BytePlus Pte Ltd., Singapore/EU hosting and the Singapore/EU compliance story. They cannot be treated as one vendor risk entry.
3. **The marketing is international-grade but the primary documentation is mixed-language.** Much of the verifiable mainland detail is Chinese-language and some mainland pages are not reliably reachable from outside China (see §14). Budget for translation and local-counsel review.

---

## 3. The Model Catalogue

⚠️ **Catalogue warning, up front:** this is the fastest-moving part of the subject. Every model name, version and date below is **date-stamped to 2026-09-16** and will be stale within months. BytePlus's own documentation repeatedly shows monthly and quarterly model refreshes. Never quote this section into a design decision without re-reading [docs.byteplus.com/en/docs/ModelArk/1330310](https://docs.byteplus.com/en/docs/ModelArk/1330310).

### 3.1 The family structure a developer must understand

There are **three distinct naming layers**, and confusing them is the single most common source of error:

| Layer | What it is | Example | Where it appears |
|---|---|---|---|
| **Brand / marketing name** | What the catalogue calls the model | "Dola Seed 2.0 Pro", "Dreamina Seedance 2.5" | Model list banners, playground, blog |
| **Model ID / version ID** | The string in the API `model` field for a serverless model | `seed-2-0-lite-260228`, `dola-seed-2-1-turbo`, `deepseek-v4-flash-260425` | API calls, pricing tables |
| **Inference endpoint ID** | A customer-created resource that fronts a (possibly fine-tuned) model | an opaque endpoint identifier you create | API calls after endpoint creation |
| **Version suffix convention** | Trailing `YYMMDD`-style date, plus `-ga` / `-preview` / `-turbo` / `-flash` / `-thinking` qualifiers | `-260425`, `-ga-260731` | Model IDs |

✅ **Verified (vendor docs, accessed 2026-09-16):** model IDs carrying date suffixes appear throughout the BytePlus documentation and pricing tables, e.g. `seed-2-0-lite-260228` in the Quick-start sample and `deepseek-v4-flash-ga-260731` in the pricing adjustment notice ([docs.byteplus.com/en/docs/ModelArk/1544106](https://docs.byteplus.com/en/docs/ModelArk/1544106)).

### 3.2 The models currently advertised on BytePlus ModelArk (2026-09-16)

Front-of-catalogue banners on the Model list page (✅ vendor docs, accessed 2026-09-16):

| Model | Advertised role | Vendor's own words |
|---|---|---|
| **Dola Seed 2.0** | Flagship general-purpose **agentic** model | "Built for complex reasoning and long-chain, multi-step task execution in the Agent era" |
| **Dreamina Seedance 2.5** | Mainline **video generation** | "30s extended storytelling, expanded multimodal references" |
| **Dola Seedream 5.0** | Leading **image generation** | "Enhanced reference consistency and improved generation quality for professional scenarios" |

The BytePlus AI model page gives a fuller dated inventory ([ai.byteplus.com/en/model](https://ai.byteplus.com/en/model), accessed 2026-09-16):

| Category | Model | "Updated" date shown |
|---|---|---|
| Seed (LLM) | Dola Seed 2.1 turbo | 2026-07-13 |
| Seed (LLM) | Dola Seed 2.0 pro | 2026-03-24 |
| Seed (LLM) | Dola Seed 2.0 lite | 2026-03-06 |
| Seed (LLM) | Dola Seed 2.0 mini | 2026-02-14 |
| Seedream (image) | Dola Seedream 5.0 Pro | 2026-07-08 |
| Seedream (image) | Dola Seedream 5.0 lite | 2026-02-24 |
| Seedream (image) | ByteDance Seedream 4.5 | 2025-11-28 |
| Seedream (image) | ByteDance Seedream 4.0 | 2025-08-28 |
| Seedance (video) | Dreamina Seedance 2.5 | 2026-08-07 |
| Seedance (video) | Dreamina Seedance 2.0 | 2026-06-22 |
| Seedance (video) | Dreamina Seedance 2.0 mini | 2026-06-15 |
| Seedance (video) | Dreamina Seedance 2.0 fast | 2026-04-14 |
| Seed Speech | Text-to-Speech (TTS) 2.0 | 2026-03-16 |
| Seed Speech | Speech-to-Text (ASR) 2.0 | 2026-03-16 |
| Digital human | Omnihuman 1.5 / 1.0 | 2025-10-20 / 2025-09-04 |
| Motion | DreamActor M2.0 | 2026-02-06 |

The ByteDance Seed research site lists the **upstream research family** with somewhat different labels — Seed2.1, Seed2.0, Seed1.8, Seed1.6, Seed1.5-VL, Seed1.5 (which it parenthesises as **Doubao-1.5-pro**), Seed Diffusion Preview; plus GenMedia (Seedance 2.5, Seedream 5.0 Pro, Seedream 5.0 Lite, Seedance 2.0, Seedance 1.5 pro, Seedream 4.5/4.0, SeedEdit 3.0, Seed3D 2.0/1.0, **BAGEL** described as open-source, Seed LiveInterpret 2.0, Seed Realtime Voice, Seed-Music, SeedRealtime, Seed Audio 1.0, Seeduplex), an AI-for-Science model (Protenix) and robotics models (Seed GR-3, Seed GR-RL) — [seed.bytedance.com/en/models](https://seed.bytedance.com/en/models) (accessed 2026-09-16). ✅ Verified as *what the vendor publishes*; ⚠️ the research site is a capability showcase, not a service commitment — a model being listed there does **not** mean it is callable on ModelArk.

### 3.3 Third-party and open-weight models hosted

✅ **Verified:** third-party models are hosted on the platform. The BytePlus ModelArk pricing page carries billing rows for **DeepSeek** models — `deepseek-v4-flash-260425`, `deepseek-v4-pro-260425` and `deepseek-v4-flash-ga-260731` — with explicit price-adjustment notices dated 2026-08-21 and 2026-08-28 ([docs.byteplus.com/en/docs/ModelArk/1544106](https://docs.byteplus.com/en/docs/ModelArk/1544106), accessed 2026-09-16).

❓ **Reported (secondary, mainland):** the 2023 launch roster integrated Baichuan Intelligence, 出门问问, Fudan MOSS, IDEA Research Institute, 澜舟科技, MiniMax and Zhipu AI models, and DeepSeek-V3.1 went live in August 2025 — see §2.2. **The 2023 roster almost certainly does not represent the 2026 catalogue.** Do not assume any of those models is still offered; verify on the current Model list page.

❌ **Rejected / not established:** there is no verified evidence that ByteDance's own **open-weight** releases (for example BAGEL, advertised as open-source on the Seed research site) are themselves served *as first-class models on ModelArk*. Open-sourcing a model and serving it on a commercial MaaS platform are separate decisions. ❓ Unverified either way.

### 3.4 Modalities

✅ **Verified from the documentation tree** (a page exists for each, which is strong evidence the capability is documented; it is weaker evidence that it is production-grade):

| Modality | Documented pages |
|---|---|
| **Text generation** | Text generation; streaming output; prefill-based response |
| **Vision / image understanding** | Image understanding; visual grounding |
| **Video understanding** | Video understanding |
| **Document understanding** | Document understanding |
| **Audio understanding** | Audio understanding |
| **Image generation** | Seedream 5.0 pro tutorial; interactive editing guide; general image generation tutorial |
| **Video generation** | Dreamina Seedance 2.5 tutorial + prompt guide; Seedance 2.0 series tutorial + prompt guide; portrait-video guide |
| **Embeddings** | Multimodal embedding |
| **Speech** | Seed Speech TTS 2.0 / ASR 2.0 (advertised on the model page; support pages were not individually verified) |
| **Reasoning** | Deep reasoning; deep reasoning (Responses API) |

### 3.5 Version pinning and how a model is called

✅ **Verified from vendor docs and the quick-start sample:**

- A developer either (a) calls a **serverless model ID** directly, or (b) creates an **inference endpoint** and passes the resulting **endpoint ID** in the `model` field. The Model list note confirms endpoints in one region cannot be called through another region's URL.
- Model IDs are **version-pinned by date suffix** (`-260425`, `-260731`) with qualifiers such as `-ga` (general availability), `-turbo`, `-mini`, `-lite`, `-pro`, `-fast`, `-thinking`.
- The quick-start sample uses the **Responses API** shape: `client.responses.create(model="seed-2-0-lite-260228", ...)`.
- The raw REST shape is the **Chat Completions** shape: `POST {base_url}/chat/completions` with `Authorization: Bearer $ARK_API_KEY` and `{"model": "<ENDPOINT_ID>", "messages": [...]}`.

⚠️ **Practical consequence:** because IDs are date-suffixed, an unpinned integration will drift. Pin the exact model ID string in configuration, treat any change as a release, and re-run evaluation on upgrade. This is exactly the discipline the repository's [fine_tuning_frameworks_comparison_guide.md](fine_tuning_frameworks_comparison_guide.md) and [llm_evaluation_frameworks_guide.md](llm_evaluation_frameworks_guide.md) describe for model change management; do not duplicate it here, use it.

### 3.6 Context windows and tokeniser

❓ **Partly verified.** A **256K-token context** is attributed to Doubao 1.6 by the 36Kr/界面新闻 report of the June 2025 FORCE conference ("支持 256K tokens 上下文"), and the BytePlus pricing table shows **prompt-length pricing tiers in K tokens**, with a visible band labelled `Prompt length [0, 256]` for `dola-seed-2-1-turbo` (✅ pricing docs, accessed 2026-09-16). That is consistent with 256K-class windows on current flagship models, but **per-model context limits, tokeniser identity and exact truncation semantics were not verified for the 2026 catalogue** — read the per-model pages before sizing a workload.

---

## 4. The Platform Capabilities

Throughout this section the label distinguishes **documented-to-exist** (a page in the vendor's own doc tree, which is strong evidence the feature was built) from **marketing bullet** (a claim on a product page with no documentation behind it).

### 4.1 Inference and the hosting model

✅ **Documented to exist:**

- **Serverless / standard inference** — "About inference endpoint", "Create standard inference endpoint".
- **Online inference (Flex)** — a distinct deployment mode page (`/ModelArk/2665244`).
- **Model Unit** — a dedicated capacity unit page (`/ModelArk/1568332`), the vendor's construct for reserved/committed throughput.
- **Batch inference** — asynchronous bulk job page (`/ModelArk/1399517`).
- **Context cache** — explicit caching page (`/ModelArk/1398933`), and pricing exposes **cache-hit input** and **cache-storage** line items (✅ pricing docs).
- **Prefill-based response** — server-side prefill of a prefix (`/ModelArk/1359497`).
- **Streaming output** (`/ModelArk/2123275`), **context management** (`/ModelArk/2123288`) and **context editing** for the Responses API (`/ModelArk/2123215`).
- **Files API** for uploading documents/files (`/ModelArk/1885708`).

⚠️ **Vendor claim, unquantified:** the mainland product page advertises "创建精调模型接入点后瞬时可用" (a fine-tuned model becomes instantly available once its endpoint is created) and minute-level scaling of a thousand-card GPU expansion — see §9. No independent measurement was found.

### 4.2 Fine-tuning and training

✅ **Documented to exist:** a **Training → Model fine-tuning** branch with a dataset-format specification page (`/ModelArk/1099461`). ⚠️ The *techniques* actually offered could not be fully enumerated — the visible doc tree shows dataset format and fine-tuning but does not, in the pages retrieved, itemise LoRA vs full-parameter SFT vs DPO/RLHF. ❓ **Partly unverified:** the precise method matrix, the base models eligible for tuning, and the price per tuning step.

For the discipline of choosing a tuning technique, cross-reference the repository's [fine_tuning_frameworks_comparison_guide.md](fine_tuning_frameworks_comparison_guide.md) — that guide owns the method comparison; this guide only records what ModelArk documents as shipped.

### 4.3 Evaluation and prompt tooling

✅ **Documented to exist** under the **PromptPilot** product area:

| Capability | Documented page |
|---|---|
| Prompt generation | `/ModelArk/1399496` |
| Prompt tuning | `/ModelArk/1399497` |
| Prompt management | `/ModelArk/1399498` |
| Scoring DSL | `/ModelArk/1399499` |
| VideoPilot (video-specific pilot tooling) | `/ModelArk/1883982`, API `/ModelArk/2085689` |
| AgentPilot SDK reference | `/ModelArk/1587837` |
| Structured output (beta) | `/ModelArk/1568221` (Responses API variant `/ModelArk/1958523`) |

⚠️ **Reading the evidence honestly:** PromptPilot's existence is verified; its **evaluation rigour** is not. A "scoring DSL" is a tool, not a benchmark, and the vendor publishes no independent validation of its scoring. Treat PromptPilot as prompt engineering infrastructure, **not** as a model-risk-management evaluation suite. The repository's [llm_evaluation_frameworks_guide.md](llm_evaluation_frameworks_guide.md) owns evaluation methodology; regulatory expectations for model validation are in [ai_governance_framework_guide.md](ai_governance_framework_guide.md).

### 4.4 Knowledge base / RAG

✅ **Documented to exist as a first-class managed feature** — one of the more complete areas of the documentation:

- Core process of knowledge Q&A (`/ModelArk/1261883`)
- Knowledge base CRUD: create / edit / list / view / delete (`1261885`–`1261889`)
- **Knowledge retrieval** API (`/ModelArk/1353778`) and **Knowledge Q&A** (`/ModelArk/1415553`)
- Document management: import, tag editing, delete (`1261890`–`1261892`)
- **Slice** (chunk) management: add / edit / view / delete (`1389894`–`1389898`)
- **Knowledge service management** and a **quota** page (`1554502`, `1554504`, `1343907`)
- Experimental version configuration (`/ModelArk/1904608`)

⚠️ **The vendor's marketing claim** on the mainland product page is a "high-performance knowledge base retrieval" and RAG capability with Toutiao/Douyin-grade plugins. ⚠️ No published recall, latency or scale figures support the word "high-performance". The existence of slice management and experimental versions is genuinely useful and suggests a real product rather than a stub.

The repository's `technology/ai_llm/rag/` guides own chunking, embedding selection, retrieval evaluation and the RAG architecture patterns. **This guide records only that ModelArk ships a managed knowledge base**; it does not re-derive RAG design.

### 4.5 Plugins, tools and the agent runtime

✅ **Documented to exist:**

- **Function call** (`/ModelArk/1262342`) — classic tool calling.
- **Cloud-deployed MCP / remote MCP** (`/ModelArk/1827534`) — Model Context Protocol support.
- **Integrate with third-party tools** (`/ModelArk/2160841`).
- **Managed Agents** — a full branch: Agent, Skills, MCP, Tools, **Tool permission policy**, cloud environments + cloud sandbox reference, **self-hosted environments** (overview, configuration and a dedicated *security* page), start/manage session, session event stream, **Vaults authentication**, upload-and-mount files, **persistent memory**, **Multi Agent** and **Define Outcome** (`2553713`–`2553731`, `2664204`–`2664206`).
- **AI App Lab** — an app creation/deployment surface (`/ModelArk/1333715`, `/ModelArk/1262002`).

⚠️ **Mainland marketing plugin claims:** the 火山方舟 product page advertises a **联网插件 (web-search plugin)** and content plugins described as the same ones used by 头条 (Toutiao) and 抖音 (Douyin), delivering "quality text, image and video content", alongside RAG. ⚠️ **Vendor claim.** The plugin naming is a mainland-page claim; the retrieval in the international doc tree is the tool/MCP path above.

🔗 **Cross-reference:** the 2026-era international surface has clearly converged on an **agent-runtime** product (session event stream, sandbox, persistent memory, multi-agent) rather than a plugin marketplace. The Chinese agent-framework landscape — Coze/扣子, Dify and the rest, including ByteDance's position in it — is owned by [china_ai_agent_frameworks.md](china_ai_agent_frameworks.md). **Do not re-derive the agent landscape here**; that guide is authoritative for it. What this guide adds is that ModelArk now ships its *own* managed agent runtime, which is a different thing from Coze and is a genuine build-vs-buy question against it.

### 4.6 Batch, caching and cost-control features (summary)

| Feature | Status | Note |
|---|---|---|
| Batch/async inference | ✅ Documented | Separate page; billing mechanics not verified here |
| Context cache | ✅ Documented | Pricing exposes cache-hit input and **cache-storage per M-token per hour** |
| Prefill-based responses | ✅ Documented | Server-side prefix prefill |
| Prompt caching discount | ✅ Verified in pricing | Cache-hit input priced far below fresh input (e.g. `$0.014` vs `$0.44`) |
| Reserved capacity | ✅ Documented as "Model Unit" | Commercial terms not verified |
| Coding subscription | ✅ Advertised | "Coding Plan" subscription page on BytePlus; mainland equivalent reported at ¥9.9 first month |
| Free tier | ⚠️ Advertised | A BytePlus Free Tier page exists; **the entitlements were not retrievable** (JS-rendered, see §14) |

### 4.7 What is documented versus what is marketing

| Claim | Verdict |
|---|---|
| Inference endpoints, batch jobs, context cache, Files API exist | ✅ Verified (documented) |
| Fine-tuning exists with a defined dataset format | ✅ Verified (documented) |
| A managed knowledge base with retrieval and Q&A exists | ✅ Verified (documented) |
| MCP and function calling are supported | ✅ Verified (documented) |
| A managed multi-agent runtime with sandboxes exists | ✅ Verified (documented) |
| "High-performance" retrieval; Toutiao/Douyin-grade plugins | ⚠️ Vendor claim, no independent measurement |
| "Minutes to scale a thousand cards"; instant availability of fine-tuned endpoints | ⚠️ Vendor claim, unmeasured |
| Doubao models are market leaders / 46.4% share | ⚠️ Vendor-reported IDC figure, source not retrieved |
| Cost reductions of 62.7% vs industry average | ⚠️ Vendor claim from a launch event |

---

## 5. The Developer Surface

This is the section an engineer needs. It records what the API actually looks like, what SDKs exist, how deployment and rate limits work, and what changes when you migrate from an OpenAI-style integration.

### 5.1 Is it OpenAI-compatible? Yes — with a caveat

✅ **Verified, with evidence rather than assertion.** The vendor documentation shows all of the following simultaneously:

1. A **Chat Completions** REST path: `POST {base_url}/chat/completions` with body `{"model": "...", "messages": [{"role": "user", "content": "hello"}]}` and header `Authorization: Bearer $ARK_API_KEY`.
2. A **Quick start** page whose language tabs include **cURL, Python, Go, Java and "OpenAI SDK"**.
3. An explicit **"Migrate to the official Ark SDK"** page (`/ModelArk/2663772`).
4. A separate **Responses API** family (an OpenAI-Responses-shaped surface) with its own migration guide (`/ModelArk/1585128`).

Sources: [Model list](https://docs.byteplus.com/en/docs/ModelArk/1330310), [Region availability](https://docs.byteplus.com/en/docs/ModelArk/2191806), [Overview](https://docs.byteplus.com/en/docs/ModelArk/1099455) (accessed 2026-09-16).

⚠️ **The caveat.** "OpenAI-compatible" here means **wire-compatible at the Chat Completions and Responses layers**, not **semantically identical**. In practice a migrating team should expect these differences: **model identifiers** are entirely different strings; a **created endpoint ID** may be what goes in the `model` field rather than a model name; **authentication** is an Ark API key, and the mainland API additionally documents a **signature-authentication path** (`/ModelArk/1465834`) which is not an OpenAI concept; **regional base URLs** replace `api.openai.com`; and **vendor-specific extensions** exist (context cache, prefill-based response, visual grounding, `-ga`/`-preview`/`-thinking` variants, Model Unit, PromptPilot).

❓ **Not verified:** a definitive published statement of the form "we are OpenAI-API-compatible". The compatibility is **demonstrated by the documentation shape** rather than declared. Note that third-party resellers commonly advertise Doubao/Ark models as OpenAI-compatible through their own gateways — that is a statement about the *reseller*, not about Ark.

### 5.2 SDKs and languages

| Surface | Detail | Evidence |
|---|---|---|
| Python | `from arkruntime import Ark` — the **official Ark SDK**; quick-start sample shown | ✅ Quick start, accessed 2026-09-16 |
| Go, Java | Language tabs on Quick start | ✅ Quick start |
| cURL | REST samples throughout | ✅ Region-availability page |
| OpenAI SDK | Language tab named "OpenAI SDK" (use the OpenAI client against the Ark base URL) | ✅ Quick start |
| Dedicated SDK reference | "Migrate to the official Ark SDK" migration page | ✅ Doc tree |
| BytePlus general SDK hub | `api.byteplus.com/api-sdk` exists but is **per-service (ECS shown by default)**; a ModelArk-specific SDK listing was not retrievable | ⚠️ Partially verified |
| OpenAI-compatible ecosystem | Any OpenAI-compatible client can be pointed at an Ark base URL — but see the caveat in §5.1 | ✅ Inferred from wire shape |

⚠️ The documentation distinguishes **"the official Ark SDK"** from OpenAI-client usage, which implies the vendor has a first-class SDK it wants you on. The **current version, release cadence and support window of that SDK were not verified**.

### 5.3 Console, playground and deployment surface

✅ **Verified to exist:**

- **Console** at `ai.byteplus.com/ark` (international) and `ark.volcengine.com/region:cn-beijing/...` (mainland), including a fine-tuning view.
- **Playground** ("Try in playground" links on the model page) and an "Ask BytePlus AI" playground entry.
- **Model activation** — the docs refer to per-region "model activation status", implying models must be activated before use in a region.
- **App Lab** for building and deploying an app on the platform.
- **PromptPilot** console surface for prompt work.

### 5.4 The endpoint concept (the thing that trips people up)

✅ **Verified:** the platform separates the **model** from the **inference endpoint**.

| Concept | What it is | Why it matters |
|---|---|---|
| Model (serverless) | A vendor-hosted model identified by a dated model ID | Zero-ops, per-token billing, shared capacity |
| Inference endpoint | A **customer-created resource** that fronts a model — possibly a fine-tuned one — with its own ID | Required for fine-tuned models; gives a stable identifier; is **region-scoped** |
| Model Unit | A capacity/throughput unit | The reserved-capacity construct |
| Online inference (Flex) | A separate online-serving mode | Different commercial/resource profile |

⚠️ **Region-scoping is the trap.** The vendor states plainly that *"An inference endpoint created in a region must be invoked using the region-specific base URL. Inference endpoints in one region cannot be called through another region's URL,"* and that *API keys and model activation status are isolated by region*. A multi-region deployment therefore requires **per-region endpoints, per-region keys and per-region activation** — design for it rather than discovering it.

### 5.5 Rate limits, quotas and burst behaviour

✅ **Verified (and unusually honest for a vendor):** the Model list page states that *"All rate limits listed in this document are theoretical maximum values which are not guaranteed, and will be affected by platform load and invocation method,"* and links a **"Best practices for handling burst traffic"** page. There is also a **knowledge-base quota** page (`/ModelArk/1343907`).

⚠️ **Read this as a design constraint, not boilerplate.** A vendor that declines to guarantee its published limits is telling you that capacity is shared and elastic. Any latency- or throughput-critical workload should be built with back-pressure, retry with jitter, queueing and a fallback path — or should buy committed capacity (Model Unit) instead of relying on serverless limits.

### 5.6 Error and retry semantics

❓ **Not fully verified.** The documentation set includes a general API reference (`/ModelArk/1511946`) and signature-authentication examples (`/ModelArk/1465834`), but a dedicated error-code/retry-semantics page was not retrieved. **Do not assume OpenAI error-code parity.** Verify the error taxonomy and idempotency behaviour during a migration spike.

### 5.7 Observability for a developer

✅ **Documented:** a **session event stream** for agents (`/ModelArk/2553725`), **context management** and **context editing** for controlling what the model sees, **Scoring DSL** for prompt evaluation, and console views for endpoints and fine-tuning jobs. ⚠️ A **distributed-tracing / metrics export** story (OpenTelemetry, log export, per-request token accounting feeds) was **not** verified. For a regulated institution this matters — see §11.4.

### 5.8 Migration path from an OpenAI-style integration

| Step | What you do | What changes | What does not change |
|---|---|---|---|
| 1 | Point the base URL at the Ark regional endpoint | `https://api.openai.com/v1` → `https://ark.ap-southeast.bytepluses.com/api/v3` (or EU) | Your HTTP client, retry wrapper, streaming parser |
| 2 | Swap the credential | OpenAI API key → Ark API key in `Authorization: Bearer` | The header name |
| 3 | Repoint the model field | `gpt-*` → an Ark **model ID** or an **endpoint ID** you create | The request shape |
| 4 | Re-baseline prompts | Prompt behaviour differs across model families; system-prompt handling and tool-call formatting may differ | Your prompt-management tooling |
| 5 | Re-run evaluation | You must, because the model changed | Your evaluation harness, if it is model-agnostic |
| 6 | Re-do data governance | New vendor, new regions, new terms, possibly new jurisdiction | Your internal policy framework |
| 7 | Add fallback | Given unguaranteed rate limits, add a second provider or model | Your resilience pattern |

🔗 The evaluation harness and the model-change discipline are owned by [llm_evaluation_frameworks_guide.md](llm_evaluation_frameworks_guide.md); the retrieval and self-hosting alternatives by `technology/ai_llm/rag/` and [ollama_xinference_localai_guide.md](ollama_xinference_localai_guide.md).

### 5.9 The developer-surface verdict

| Question | Answer | Confidence |
|---|---|---|
| OpenAI-compatible? | Yes at the Chat Completions / Responses wire level; not declared, and not semantic | ✅ High |
| Official SDKs? | Python (`arkruntime`), plus Go/Java/cURL samples; OpenAI SDK usable | ✅ High |
| Endpoint/region complexity? | Real and non-trivial: per-region endpoints, keys and activation | ✅ High (vendor-documented) |
| Rate limits guaranteed? | **No — the vendor says they are not** | ✅ High |
| Error/retry taxonomy documented? | Not retrieved | ❓ Low |
| Observability export documented? | Not retrieved | ❓ Low |
| Migration effort from OpenAI-style | Low for the wire layer, **high for governance, prompts, evaluation and residency** | ✅ Medium-high |

---

## 6. The Commercial Model and the Pricing

⚠️ **Read this section as a snapshot, not a price list.** Prices on this platform change by announcement, sometimes within weeks (the vendor published *two* adjustments in August 2026 alone). Every number below is **as published on 2026-09-16** and every one of them must be re-checked before it enters a business case.

### 6.1 The structure

✅ **Verified from the BytePlus ModelArk pricing page** ([docs.byteplus.com/en/docs/ModelArk/1544106](https://docs.byteplus.com/en/docs/ModelArk/1544106), accessed 2026-09-16):

The billing table is organised by **model ID** with these columns:

| Column | Meaning |
|---|---|
| Model ID | The exact dated model identifier |
| Pricing tiers (K tokens) | **Input-length bands** — e.g. `[0, 256]` K tokens |
| Input (non-audio) USD/M tokens | Fresh input price |
| Input (audio) USD/M tokens | Audio input priced separately where applicable |
| **Cache-storage** USD/M tokens/Hour | A charge for *holding* cached context |
| **Cache-hit input (non-audio)** USD/M tokens | Discounted price for cached input |
| Cache-hit input (audio) USD/M tokens | Discounted cached audio input |
| Output USD/M tokens | Generation price |

Four structural features matter commercially:

1. **Input/output split** — output tokens are priced far above input (see the examples below).
2. **Input-length tiering** — price rises with prompt length, so long-context workloads cost more per token than short ones. This is a deliberate design (see §6.3).
3. **Cache-hit discount** — cached input is dramatically cheaper than fresh input.
4. **Cache-storage time charge** — you pay *per hour* for retained cache, which means a naive caching strategy can cost money even when nothing is being generated.

### 6.2 Published unit prices (BytePlus ModelArk, accessed 2026-09-16)

The vendor published explicit price-adjustment notices:

| Model ID | Adjustment effective (UTC+8) | Input (non-audio) | Output | Cache-hit input (non-audio) |
|---|---|---|---|---|
| `deepseek-v4-flash-260425` | 2026-08-28 00:00 | $0.44 | $1.32 | $0.014 |
| `deepseek-v4-pro-260425` | 2026-08-28 00:00 | $1.32 | $3.96 | $0.044 |
| `deepseek-v4-flash-ga-260731` | 2026-08-21 00:00 | $0.44 | $1.32 | $0.014 |

✅ All figures are **USD per million tokens** as printed on the vendor's page. A representative first-party row from the online-inference (standard) table:

| Model ID | Tier | Input (non-audio) | Cache-storage /M/hour | Cache-hit input | Output |
|---|---|---|---|---|---|
| `dola-seed-2-1-turbo` | Prompt length [0, 256]K | $0.50 | $0.0083 | $0.10 | $2.50 |

⚠️ **Textbook reading of that row:** cache-hit input is **5× cheaper** than fresh input ($0.10 vs $0.50) but cache storage is billed **hourly**, and output is **5× the input price**. Any cost model that only multiplies tokens by one number is wrong on this platform.

❓ **Not verified:** the complete price table (the page is long and JS-assisted), the **Model Unit / reserved-capacity** price, **fine-tuning** prices, **batch inference** discounts, **knowledge-base** and **vector-storage** prices, **image/video generation** prices (Seedance/Seedream are usually priced per generation or per second, not per token), and the **Coding Plan** subscription price. The BytePlus *Coding Plan* activity page exists and is *marketed* from $4.50/month in third-party write-ups — ⚠️ **that figure comes from a third-party gist, not the vendor page, and is not verified.**

### 6.3 The price-war history (verified events, vendor-reported numbers)

The Chinese model price war is the context that explains the pricing structure. Reconstructed from third-party business press and vendor statements:

| Date | Event | Price detail | Source quality |
|---|---|---|---|
| **2024-05-15** | 火山引擎 FORCE conference: Doubao family opened externally at "厘" (sub-cent) pricing | Doubao claimed **>99% cheaper** than the then-industry position for a family flagship | ❓ Reported by [知乎 FORCE write-up](https://zhuanlan.zhihu.com/p/698265798); the 99% figure is described in 36Kr as 谭待's own statement (⚠️ vendor claim) |
| **2025-06-11/12** | FORCE conference: **Doubao 1.6** + **Seedance 1.0 pro** | **Input-length tiered pricing** introduced. 0–32K band: **¥0.80 / M input, ¥8 / M output** — **63% below Doubao 1.5**; 32K–128K: ¥1.2 / ¥16; 128K–256K: ¥2.4 / ¥24. Seedance 1.0 pro at **¥0.015 per 1K tokens**, ≈ **¥3.67 per 5-second 1080p video**, ≈70% below comparable video models | ✅ [36Kr reprint of 界面新闻, 2025-06-12](https://m.36kr.com/p/3333345351248136) — third-party business press, detailed and specific |
| **2025-06** (same event) | Volume context | Doubao **daily tokens > 16.4 trillion**, **137× the May 2024 launch level**; IDC: Doubao **46.4%** of China's public-cloud LLM market | ⚠️ **Vendor-reported** via press. The IDC report itself was not retrieved |
| **2025-11-11** | Doubao-Seed-Code released | Vendor claims comprehensive cost **62.7% below industry average**, "lowest in China"; Coding Plan from **¥9.9 first month** | ❓ Secondary (百家号 via Baike); ⚠️ vendor claim |
| **2026-08-21 / 2026-08-28** | BytePlus ModelArk adjustments on DeepSeek-family rows | Prices as tabulated in §6.2 | ✅ Vendor pricing page, accessed 2026-09-16 |

⚠️ **The critical reading:** the *events* (launches, the introduction of input-length tiering, the August 2026 adjustments) are **documented**. The *comparative percentages* ("63% below", "99% cheaper", "62.7% below industry average", "70% below") are **vendor claims**, usually made on stage and repeated by press. No independent cost benchmarking was found. Treat them as marketing positioning with a real underlying price cut, not as measured savings.

### 6.4 Free, beta and enterprise allowances

| Item | Status |
|---|---|
| BytePlus Free Tier | ✅ A page exists at `byteplus.com/activity/free`; ❌ **the actual entitlements were not retrievable** (the page is JS-rendered and returned only navigation chrome) |
| Coding Plan subscription | ✅ Advertised (BytePlus activity page; mainland equivalent reported) — ❓ price not verified |
| Beta features | ✅ "Structured output (beta)" is labelled beta in the doc tree — a reminder that some capabilities are pre-GA |
| Enterprise contracting | ❓ **Not verified.** BytePlus/Volcengine enterprise agreements, committed-spend discounts, Model Unit pricing and negotiated terms are not public |

### 6.5 Cost-control features a bank should use

| Feature | Effect | Verified? |
|---|---|---|
| **Context cache** | Cache-hit input at a large discount to fresh input | ✅ Pricing exposes the discount |
| **Cache-storage awareness** | Hourly storage charge means cache lifetime must be tuned | ✅ Pricing exposes the charge |
| **Input-length tiering** | Keep prompts in the cheapest band; long prompts are disproportionately expensive | ✅ Pricing structure |
| **Model right-sizing** | `-mini` / `-lite` / `-flash` variants exist at lower tiers | ✅ Catalogue |
| **Batch inference** | Async bulk path (discount not verified) | ✅ Documented, ❓ priced |
| **Model Unit** | Reserved capacity, potentially cheaper at steady state | ✅ Documented, ❓ priced |
| **Prompt-prefix discipline** | Stable prefix ordering maximises cache hits | ⚠️ Inferred from the cache product, not from a published best-practice doc |

### 6.6 The commercial verdict

| Statement | Verdict |
|---|---|
| Per-token, per-model pricing with input/output split and length tiers | ✅ Verified |
| Cache discounts and hourly cache-storage charges exist | ✅ Verified |
| Two price adjustments in August 2026 alone | ✅ Verified |
| The platform is positioned as a low-price leader | ✅ Verified as positioning; ⚠️ the percentage claims are vendor claims |
| Full price list, tuning prices, capacity prices, free-tier entitlements | ❌ Not verified |
| Prices are materially cheaper than Western equivalents on a like-for-like basis | ❓ **Not established.** Currency, tier, model class, caching and egress all move the answer; no independent benchmark was found |

---

## 7. The Compliance and Data-Governance Position

This is the section that decides whether a bank can use the platform at all. It is also the section where the two variants diverge most sharply, because **they answer to different regulators**.

### 7.1 The two regimes, separated

| Dimension | Mainland China (火山方舟) | International (BytePlus ModelArk) |
|---|---|---|
| Contracting entity | Chinese operating entity of 火山引擎 | **BytePlus Pte Ltd.** (Singapore) |
| Governing law | PRC law | Contractual; BytePlus documents alignment with **Singapore PDPA** and **EU GDPR** |
| AI-service filing | **YES — 生成式人工智能服务备案 applies** | Not applicable; no PRC filing regime |
| Hosting regions | Mainland Chinese regions (e.g. `cn-beijing`) | `ap-southeast-1` (Johor, Malaysia), `eu-west-1` (Dublin, Ireland) |
| Data-protection posture | PRC PIPL / DSL / CSL | PDPA / GDPR-aligned |
| Who can realistically buy it | An entity with a Chinese presence and a Chinese use case | A non-Chinese entity, or a Chinese entity's offshore operation |

✅ **Verified:** the BytePlus entity and the PDPA/GDPR statement are on [byteplus.com/en/trust-center](https://www.byteplus.com/en/trust-center); the international regional endpoints are on [Region availability](https://docs.byteplus.com/en/docs/ModelArk/2191806), and the mainland console region is visible in `ark.volcengine.com/region:cn-beijing/...` (all accessed 2026-09-16).

### 7.2 The Chinese filing regime — verified from the regulator's own site

✅ **Verified from the Cyberspace Administration of China (CAC / 国家网信办)**, announcement *"关于发布生成式人工智能服务已备案信息的公告（2025年7月至8月）"*, published 2025-09-10 ([cac.gov.cn](https://www.cac.gov.cn/2025-09/10/c_1759222982377536.htm), accessed 2026-09-16). The announcement states, in the regulator's own words, that:

- Cyberspace authorities, with other departments, **continue to carry out 备案 (filing) for generative AI services in accordance with the 《生成式人工智能服务管理暂行办法》** — the **Interim Measures for the Management of Generative AI Services**.
- In **July–August 2025, 99 new generative-AI services completed national-level 备案**.
- **Applications or functions that call an already-filed model's capabilities through an API or otherwise are handled by local cyberspace authorities as 登记 (registration)**, and **27 new registrations** were completed in that period.
- **As of 2025-08-31, cumulatively 538 generative-AI services had completed 备案 and 263 applications/functions had completed 登记.**
- Providers of generative-AI services **with public-opinion attributes or social-mobilisation capability** file or register through the **local (属地) cyberspace authority**.
- **Live generative-AI applications must publicly display, in a prominent position or on the product detail page, the filed/registered service used, giving the model name and the 备案号 (filing number) or launch number.**

⚠️ **Why this matters operationally:** the filing obligation attaches to the **service provider**, not to the enterprise calling the API. If a mainland business deploys an application on top of an already-filed model, its obligation is the locally administered **登记** — and it must publish the model name and filing number on the product. **Design the compliance disclosure into the product from day one**; it is not a back-office task.

❓ **What this guide could NOT establish (and this is important):**

1. **Whether 豆包 / 火山方舟 specifically appears in the CAC register, and under what filing number.** The published filing lists are attachments to separate CAC announcements, and the attachment content was **not parsed in this research**. The platform and the models *almost certainly* hold filings — a service of this scale could not operate in mainland China otherwise, and the vendor's marketing implies it — but ⚠️ **"almost certainly" is not verification.** **Before relying on this, retrieve the CAC filing list and find the exact entry.** This is a five-minute check for someone inside China and an important one.
2. **The Interim Measures' own effective date.** The CAC announcement confirms the instrument is operative; its commencement date was not verified from a primary CAC page here. ❓ Widely reported as August 2023 — treat as reported, not verified.
3. **Third-party aggregate filing counts are unreliable and mutually inconsistent.** Three separate secondary sources claim different totals — **611 by 2026-03**, **748 cumulative by end-2025**, and **796 cumulative** in a 2026 developer-community article. ❌ **Reject all three as authoritative.** The CAC's own figure of **538 as of 2025-08-31** is the only primary number retrieved; any later total requires reading the subsequent CAC announcements.

### 7.3 What the vendor says it does with your data

⚠️ **Vendor claim, read directly from the vendor's own announcement (BytePlus blog, 2026-03-24):** *"BytePlus is committed to protecting customer data entrusted to its platform. Data is processed strictly based on customer instructions, with strong safeguards in place to prevent unauthorized access. BytePlus does not access customer data without explicit permission, and each customer's data is logically isolated and not shared with third parties unless authorized."* — [byteplus.com/en/blog/dola-seed-2-0-pro](https://www.byteplus.com/en/blog/dola-seed-2-0-pro) (accessed 2026-09-16).

On the mainland side, ⚠️ **vendor claims** reported from the product page include: a "mutual-trust computing security architecture" (互信计算安全架构) for model-and-data lifecycle security; protection of model providers' intellectual property; a network architecture described as transparent, controllable and auditable (访问可控可审计); **安全防护栏 (safety guardrails)** providing content-risk identification and **智能脱敏 (intelligent de-identification/masking)**; a **transparently encrypted file system** where data is encrypted on write (数据落盘即加密); and encrypted training using distance-preserving encryption with differential privacy (保距加密和差分隐私). — [volcengine.com/product/ark](https://www.volcengine.com/product/ark) / Baidu Baike summary of that page (accessed 2026-09-16).

❌ **NOT verified — and do not assume:** the actual **contractual** terms. This research did **not** read the BytePlus **Terms of Service** or **Privacy Policy** (`docs.byteplus.com/en/legal/...`) or the mainland service agreement line by line. **"Does the vendor train on my inputs?" is a question that is answered in those documents, not in a blog post.** A blog commitment is a statement of intent, not a contractual warranty, and it can be changed. **A regulated buyer must have counsel read the executed terms before any customer data is sent.**

### 7.4 Data residency and deployment options

✅ **Verified:** region-scoped inference endpoints; region-specific base URLs; region-isolated API keys and model activation (`ap-southeast-1` and `eu-west-1` internationally).

⚠️ **The residency caveat that actually matters.** The vendor's own Region availability page states that while inference requests to an endpoint *"are primarily routed to the region where the endpoint is created"*, **"based on overall resource scheduling, some requests may be routed to inference resources in other regions."** That is a documented possibility of **cross-region processing**. For a bank with a hard residency requirement, this single sentence must be resolved with the vendor in writing — either by contractual commitment, by a configuration that disables cross-region routing, or by accepting it. ❓ **Unclear whether a customer can opt out.** Flag as a **must-ask**.

| Deployment question | Finding |
|---|---|
| Region selection | ✅ Yes — two international regions; multiple mainland regions |
| Regional isolation of endpoints/keys | ✅ Documented |
| Guarantee that data never leaves the chosen region | ⚠️ **Not guaranteed** — the docs admit possible cross-region scheduling |
| Private / dedicated / VPC deployment of the *model service* | ❓ **Not verified.** No dedicated-VPC or single-tenant model-serving product was documented in the pages retrieved |
| Self-hosted option | ⚠️ Partially: **self-hosted environments for Managed Agents** exist (documented, with a security page) — but this self-hosts the *agent execution environment*, **not** the model inference |
| On-premises model deployment | ❌ Not offered by this platform; that is a different product category (see [ollama_xinference_localai_guide.md](ollama_xinference_localai_guide.md)) |

### 7.5 Content security and moderation posture

⚠️ **Vendor claims:** safety guardrails with content-risk identification and intelligent de-identification (mainland product page); "content moderation" as a supported enterprise use case for the flagship model (BytePlus blog). ✅ The **existence** of a moderation-oriented capability is plausible and consistent with the PRC regulatory environment, where content security is mandatory. ❓ However, **no API-level moderation endpoint, policy taxonomy, rate of false positives, appeal process or published moderation policy** was verified. For a bank this means: **you cannot rely on the platform's moderation as your control**; you must implement your own input/output filtering regardless.

### 7.6 The compliance verdict

| Question | Answer |
|---|---|
| Is there a filing regime in mainland China? | ✅ **Yes** — CAC 备案 under the Interim Measures (primary source retrieved) |
| Does an application built on a filed model need its own registration? | ✅ **Yes** — local 登记, and it must publish the model name + 备案号 |
| Does 豆包/火山方舟 hold a filing? | ❓ **Not verified in this research** — the register entry was not located. **Check this; it is the single highest-value open question** |
| Are the international and mainland variants the same compliance proposition? | ❌ **No.** Different entity, law, regions and regime |
| Do the vendor's blog commitments constitute contractual protection? | ❌ **No.** Read the executed terms |
| Is cross-region processing possible? | ⚠️ **Documented as possible.** Must be contractually resolved |
| Is a private/VPC model-serving deployment available? | ❓ **Not verified** |

---

## 8. The Security Posture

### 8.1 Isolation model

| Layer | Finding | Quality |
|---|---|---|
| Tenant data isolation | "Each customer's data is **logically isolated** and not shared with third parties unless authorized" | ⚠️ Vendor claim (BytePlus blog) |
| Regional isolation | Endpoints, API keys and model activation are **region-isolated** | ✅ Vendor-documented behaviour |
| Model-provider isolation | Mainland page claims a mutual-trust computing architecture protecting model providers' IP | ⚠️ Vendor claim |
| Single-tenant / dedicated tenancy | ❌ Not documented | ❓ |
| Encryption at rest | Mainland page claims a transparently encrypted file system, encrypted on write | ⚠️ Vendor claim |
| Encryption in transit | ❌ Not verified in the pages retrieved (HTTPS is implied by the HTTPS base URLs) | ❓ |

⚠️ **"Logical isolation" is a meaningful but bounded statement.** It normally means shared infrastructure with per-tenant logical separation in the application and data layers — not dedicated hardware and not cryptographic separation per tenant. A bank should ask whether "logical" here is at the row, table, database or account boundary, and what the vendor's own administrative access controls are.

### 8.2 Authentication and authorisation

✅ **Verified:** an API key presented as `Authorization: Bearer $ARK_API_KEY`, with **region-isolated keys** and per-region **model activation status**. ✅ Additionally documented: a **signature authentication** scheme with examples (`/ModelArk/1465834`) — a separate, non-Bearer authentication path — and **Vaults authentication** for managed-agent sessions (`/ModelArk/2553726`). ✅ A **tool permission policy** exists for agent tools (`/ModelArk/2553720`), which is a genuine authorisation control rather than a marketing word.

❓ **Not verified:** IAM granularity (roles, service accounts, least-privilege scoping), key rotation policy, key-expiry defaults, whether keys can be scoped per endpoint or per model, and whether hardware-backed key management (HSM/KMS) is available for customer-managed keys. **No customer-managed-key (BYOK) claim was found.**

### 8.3 Network and VPC options

❓ **Not verified.** The mainland product page claims a "transparent network architecture" that is "access-controllable and auditable" (网络架构透明化，访问可控可审计). No **private-link / VPC endpoint / IP allow-listing / dedicated interconnect** documentation was retrieved for the model service. For a bank, private connectivity is usually a hard requirement, and **its absence from the retrieved documentation is a finding in itself** — either it exists and is under-documented, or it does not exist. Ask.

### 8.4 Audit and observability for a security team

✅ **Documented:** in-product audit surfaces implied by "auditable access"; per-region activation state; a **session event stream** for agent runs (`/ModelArk/2553725`); **Tool permission policy** as an enforceable control.

❓ **Not verified:** the existence of **API call logging / access logs exportable to a SIEM**, administrator-action audit trails, retention configuration, or any compliance-oriented audit report generated for the customer. **For a regulated institution, "can we feed this into our SIEM?" is a pass/fail question, and the answer was not established here.**

### 8.5 Certifications and attestations

✅ **Verified from the vendor's own trust centre** for **BytePlus** (the international entity): ISO 9001, ISO 20000, ISO 22301, **ISO 27001**, **ISO 27017**, **ISO 27018**, **ISO 27701**, **MTCS Level 3**, **CSA STAR**, **SOC 1**, **SOC 2**, **SOC 3**, **PCI DSS**, plus **Singapore DPTM** and **APEC CBPR / APEC PRP**. The page also states alignment with Singapore PDPA and EU GDPR ([byteplus.com/en/trust-center](https://www.byteplus.com/en/trust-center), accessed 2026-09-16).

| Certification | What it evidences | Caveat |
|---|---|---|
| ISO 27001 | An ISMS exists and is audited | Scope (which services, which regions) not verified |
| ISO 27017/27018/27701 | Cloud security, PII in cloud, privacy management extensions | As above |
| SOC 1 / 2 / 3 | Controls relevant to financial reporting / security / public summary | **Reports are available under NDA to customers**; not public. SOC 3 is the public one |
| PCI DSS | Payment-card environment controls | Relevance to a model API is limited; scope unverified |
| MTCS Level 3 / DPTM / APEC CBPR/PRP | Singapore and APEC-region data-protection standing | Useful if the bank's data is Singaporean |

⚠️ **Two important limitations:** (1) these certifications belong to **BytePlus**, not to the mainland 火山引擎 platform — **they do not transfer automatically to a mainland deployment**; (2) a certification badge on a marketing page is **not** an audit report. A regulated buyer must request the actual **SOC 2 Type II report** and the certification scope statements under NDA. The mainland platform separately claims SOC 2 and ISO-series assessment (❓ secondary source, Baidu Baike summarising the product page — **not verified against a certificate**).

### 8.6 The "session trace-free" / 不留痕 style claim

❓ **NOT VERIFIED — and this guide deliberately does not assert it.** The task brief referenced a reported "session trace-free" / 不留痕 style claim about session logging. **No primary vendor page containing such a claim was retrieved in this research.** The closest verified statements are the weaker, generic ones quoted in §7.3 (data processed on instruction; no access without permission; logical isolation).

**What such a claim would and would not mean if it existed:** "trace-free" in Chinese cloud marketing typically means *the vendor does not retain the conversation content of a session for its own purposes* — it is a statement about **retention and secondary use**, not about encryption, not about isolation, and **not** a statement that no logs exist. It would not, for example, preclude abuse-prevention logging, billing-metadata retention, or legally compelled disclosure. ⚠️ **Any bank that finds such a claim must convert it into a contractual term with a defined retention period, defined metadata fields and a defined deletion SLA.** An adjective is not a control.

### 8.7 The security verdict

| Control area | Status |
|---|---|
| Identity and API authentication | ✅ Documented (Bearer key, signature auth, agent vaults) |
| Agent tool authorisation | ✅ Documented (tool permission policy) |
| Tenant isolation | ⚠️ Vendor claim ("logical isolation"), scope unverified |
| Encryption at rest | ⚠️ Vendor claim (mainland page) |
| Encryption in transit / key management / BYOK | ❓ Not verified |
| Private network connectivity | ❓ Not verified |
| Customer-visible audit logging / SIEM export | ❓ Not verified |
| Third-party certifications | ✅ Verified for the international entity; ⚠️ not transferable to the mainland platform |
| Session non-retention ("trace-free") | ❓ **Unverified claim; not asserted here** |

---

## 9. The Scale and Performance Claims

The rule for this section: **the source of a claim determines its evidential value.** A vendor statement is a statement about intent and capability. It is not a benchmark. Where no independent measurement exists, this section says so rather than implying one.

### 9.1 The claims, as made

| Claim | Exact framing | Source | Verdict |
|---|---|---|---|
| Abundant public-cloud GPU resource pool | "充沛的公有云GPU资源池" | 火山方舟 product page (mainland) | ⚠️ Vendor claim, unquantified |
| Stability under very large traffic | "保障超大流量下的业务稳定，算力充足功能完备" | Same | ⚠️ Vendor claim, no SLO published |
| Fine-tuned model endpoint usable instantly | "创建精调模型接入点后瞬时可用" | Same | ⚠️ Vendor claim; "instant" is undefined |
| Thousand-card scale-out in minutes | "分钟级完成千卡扩缩容" | Same | ⚠️ Vendor claim; no time series, no percentile |
| Daily token volume | Doubao **>16.4 trillion tokens/day**, **137×** the May 2024 launch level | 火山引擎 reported via [36Kr/界面新闻 2025-06-12](https://m.36kr.com/p/3333345351248136) | ⚠️ Vendor-reported. Internally consistent as a growth story; **not independently measured** |
| Market share | Doubao **46.4%** of China's public-cloud LLM market, #1 | IDC, reported via the same article | ⚠️ **Second-hand vendor-favourable statistic.** The IDC report itself was not retrieved |
| Internal validation | Models validated across **50+ internal ByteDance scenarios** | [volcengine.com/product/doubao-dy](https://www.volcengine.com/product/doubao-dy) | ⚠️ Vendor claim |
| Cost leadership | Several percentage claims (62.7%, 63%, 99%, 70%) | Vendor statements at launch events | ⚠️ Vendor claims — see §6.3 |

### 9.2 The honest reading of each

- **"Abundant GPU pool" and "minutes to a thousand cards"** are plausible for a hyperscaler of ByteDance's size — the company runs one of the largest content-serving estates in the world, and the marginal cost of a GPU pool is something it genuinely has. But **plausibility is not measurement.** What is missing is the thing an architect needs: **p50/p99 latency at a stated concurrency, sustained throughput per endpoint, and time-to-first-token distribution**. None of those figures was published in any source retrieved.
- **"137× growth" and "16.4 trillion tokens/day"** are compelling *capacity* evidence in one narrow sense: a platform genuinely handling trillions of tokens a day is a platform that does not melt under load. They say nothing about **your** latency at **your** concurrency. They are also self-reported and unaudited.
- **"46.4% market share"** is the claim with the weakest sourcing. It is attributed to IDC but was encountered only through a press article. ❓ **Unverified.** A bank should not repeat this number without the underlying IDC document.
- **The vendor's own documentation contradicts the aspirational tone of the marketing in one useful way.** The Model list page states plainly that *"All rate limits listed in this document are theoretical maximum values which are not guaranteed, and will be affected by platform load and invocation method."* ✅ **Verified.** That is the single most decision-relevant performance statement in the entire documentation set, and it is a **negative** one.

### 9.3 What independent measurement exists

❓ **None was found in this research.** No published third-party benchmark of ModelArk / 火山方舟 latency, throughput, availability or error rates was located. The sources that do discuss performance — API aggregators, reseller sites, AI-directory blogs and gist write-ups — are **promotional or affiliate-driven** and none presented methodology. Structural note for the reader: a large fraction of the "independent" material about Chinese model APIs is SEO content built on the vendor's own numbers.

⚠️ **Therefore:** treat the entire performance dossier as **vendor claims plus one negative vendor-documented caveat.** The correct posture for a bank is a **load test against your own workload before commitment**, with contractual service levels attached — because the published limits are explicitly not guaranteed.

### 9.4 The scale verdict

| Question | Answer |
|---|---|
| Does the vendor make large capacity claims? | ✅ Yes, and they are plausible |
| Are they quantified with methods? | ❌ No |
| Is there independent measurement? | ❌ **None found** |
| Are rate limits guaranteed? | ❌ **No — the vendor says they are not** |
| Does the platform demonstrably carry enormous production volume? | ⚠️ Very likely on the vendor's own numbers; unaudited |
| Should a bank size this on vendor claims? | ❌ **No — load-test and contract for SLAs** |

---

## 10. The Competitive Position

### 10.1 The competitive set, verified

| Platform | Vendor | Verified identity | Source |
|---|---|---|---|
| **Volcano Ark / 火山方舟** (CN) · **ModelArk** (intl) | ByteDance — Volcengine / BytePlus | ✅ | volcengine.com/product/ark; byteplus.com/en/product/modelark |
| **Amazon Bedrock** | AWS | ✅ | Covered in [enterprise_ai_platforms_guide.md](enterprise_ai_platforms_guide.md) |
| **Azure AI / Azure AI Foundry** | Microsoft | ✅ | Same |
| **Vertex AI** | Google Cloud | ✅ | Same |
| **阿里云百炼 (Model Studio)** | Alibaba Cloud | ✅ Verified — "大模型服务平台百炼…一站式提供丰富的通义大模型与灵活的智能应用开发工具" | [aliyun.com/product/bailian](https://www.aliyun.com/product/bailian), docs at help.aliyun.com/zh/model-studio (accessed 2026-09-16) |
| **腾讯混元大模型 (Tencent Hunyuan / HY)** | Tencent Cloud | ✅ Verified — a full-model-stack family covering text, multimodal, image generation, plus the 腾讯元器 agent platform and AI-search plugins | [cloud.tencent.com/product/tclm](https://cloud.tencent.com/product/tclm) (accessed 2026-09-16) |
| **百度智能云千帆大模型平台 (Baidu Qianfan)** | Baidu AI Cloud | ✅ Verified — an enterprise one-stop model and application development platform; ERNIE 5.1 is the current 文心 generation | [cloud.baidu.com/product-s/qianfan_home](https://cloud.baidu.com/product-s/qianfan_home) (accessed 2026-09-16) |

⚠️ **One live caveat found:** Tencent's own documentation page notes that some Hunyuan capabilities "will gradually migrate to **TokenHub**" and that the original platform will stop accepting new [customers/onboarding]. ❓ Partially read; **if Tencent is a comparator in your evaluation, verify the current product name and migration status directly.** This is a good illustration of why the naming discipline in §1 matters for *every* vendor in this space, not just ByteDance.

### 10.2 The positioning table

Rating scale: ✓✓ strong · ✓ adequate · ~ mixed/unclear · ✗ weak or unavailable. Assessments are this guide's analytical judgement from the evidence in §§3–9, not vendor claims.

| Axis | ModelArk / 火山方舟 | AWS Bedrock | Azure AI | Vertex AI | Alibaba Bailian | Tencent Hunyuan | Baidu Qianfan |
|---|---|---|---|---|---|---|---|
| First-party frontier model | ✓✓ (Dola Seed / Doubao) | ✗ (no first-party frontier LLM of its own) | ✓ (via OpenAI partnership) | ✓ (Gemini) | ✓✓ (Qwen/通义) | ✓ (Hunyuan) | ✓✓ (ERNIE/文心) |
| Third-party model breadth | ✓ (DeepSeek and others on the platform) | ✓✓ (broadest catalogue) | ✓✓ (broad) | ✓✓ (broad) | ✓✓ (broad, incl. open-weight) | ✓ | ✓ |
| Multimodal **generation** (image/video) | ✓✓ (Seedream, Seedance, Dreamina) | ~ (partner models) | ~ (DALL·E partner, video limited) | ✓ (Veo/Imagen) | ✓ (Wan/通义万相) | ✓✓ (Hunyuan image/video) | ✓ |
| Agent runtime **on the model platform** | ✓✓ (Managed Agents: sandbox, memory, MCP, multi-agent) | ✓✓ (Bedrock Agents / AgentCore class) | ✓✓ (Foundry agents) | ✓✓ (Vertex Agent Engine) | ✓ (Bailian app/agent tooling) | ✓ (元器 / TokenHub) | ✓✓ (Agent-centric positioning) |
| RAG / knowledge base as a managed feature | ✓ (documented, full CRUD + retrieval API) | ✓✓ | ✓✓ | ✓✓ | ✓✓ | ✓ | ✓✓ |
| Price aggressiveness | ✓✓ (documented cuts; low published unit prices) | ~ | ~ | ~ | ✓ (Chinese price competition) | ✓ | ✓ |
| Multi-region / global footprint | ✗ (two international regions + mainland) | ✓✓ (global) | ✓✓ (global) | ✓✓ (global) | ~ (primarily China + some intl) | ~ (primarily China) | ~ (China + intl. site) |
| PRC 备案 filing regime | ✓✓ (in-regime, mainland) | ✗ (would need a Chinese partner/model) | ~ | ✗ | ✓✓ | ✓✓ | ✓✓ |
| Western certifications (ISO/SOC scope for the AI service) | ✓ for BytePlus; ❓ for mainland | ✓✓ | ✓✓ | ✓✓ | ✓ | ✓ | ✓ |
| Enterprise governance tooling (policy, guardrails, cost controls, audit export) | ~ (guardrails and tool-permission policy claimed; audit export unverified) | ✓✓ | ✓✓ | ✓✓ | ✓ | ✓ | ✓ |
| English-language documentation depth | ✓ (ModelArk docs are complete in English) | ✓✓ | ✓✓ | ✓✓ | ~ (much in Chinese) | ~ | ~ |
| Independent third-party benchmarking | ✗ (none found) | ✓✓ | ✓✓ | ✓✓ | ~ | ~ | ~ |
| Data-residency guarantee | ~ (documented possibility of cross-region routing) | ✓✓ (strong regional guarantees) | ✓✓ | ✓✓ | ~ | ~ | ~ |

### 10.3 The honest reading of the table

**Where ModelArk is genuinely strong:**

1. **First-party multimodal breadth.** Owning both the frontier LLM and the image/video generation models (Seedream, Seedance, Dreamina) is an advantage Azure and AWS do not have — they depend on partners. For a Chinese-language, media-rich workload this is the headline differentiator.
2. **Agent runtime integrated into the model platform.** Sandbox, persistent memory, MCP, multi-agent, tool permission policy and session event streams are a coherent 2026-era agent platform, not a bolt-on. This is *not* marketing-only; it is documented in depth.
3. **Price.** Documented, dated price reductions plus a published structure with cache discounts.
4. **Documentation completeness in English.** The ModelArk doc tree is genuinely navigable in English, which is not true of all Chinese competitors. (⚠️ The mainland 火山方舟 documentation is largely Chinese and was partly unreachable from this host — see §14.)

**Where it is genuinely weak — and these are decision-relevant:**

1. **No independent benchmarking.** For every Western comparator there is a body of third-party testing; for this platform there is none.
2. **Two international regions only.** A global enterprise cannot build a worldwide deployment on two regions and expect parity with Bedrock's footprint.
3. **Data residency has a documented soft spot** — the vendor admits possible cross-region routing.
4. **Enterprise governance instrumentation is unverified.** No SIEM export, no BYOK, no verified private networking, no verified audit trail. In a regulated institution these are pass/fail.
5. **PRC/AI filings are unverified for the specific service** (§7.2).
6. **Certifications attach to BytePlus, not to the mainland platform.**

### 10.4 For whom does this platform make sense?

| Buyer | Verdict | Reasoning |
|---|---|---|
| **A Chinese-language-first consumer or media product, mainland or APAC** | ✅ **Strong fit** | Best price/quality access to Chinese-language frontier models, native multimodal generation, in-regime filings |
| **An APAC enterprise, non-regulated, wanting Chinese models with English docs** | ✅ **Good fit** | `ap-southeast-1` region, English documentation, broad model choice |
| **An AI-first startup optimising cost** | ✅ **Good fit** | Published low prices, cache discounts, Coding Plan |
| **A European enterprise with EU data-residency obligations** | ~ **Conditional** | `eu-west-1` exists, but the cross-region routing statement and unverified private networking must be resolved |
| **A regulated institution needing on-prem/private deployment** | ❌ **Poor fit** | No dedicated-tenancy or private deployment of model inference was documented |
| **A regulated institution requiring customer-managed keys and SIEM audit export** | ~ **Unproven** | Neither was verified; the burden of proof is on the vendor |
| **An enterprise with a single-vendor prohibition** | ❌ **Poor fit** | The whole stack — models, inference, agents, retrieval, coding tool — is ByteDance |
| **A Western bank's existing Bedrock/Azure/Vertex estate** | ~ **Niche** | Justifiable only where a specific Chinese-language or China-market requirement exists that the incumbent cannot meet |

🔗 **Do not re-derive the evaluation machinery here.** The peer-by-peer evaluation criteria, the scoring frameworks, the governance overlays and the vendor-lock-in analysis live in [enterprise_ai_platforms_guide.md](enterprise_ai_platforms_guide.md). That guide owns those frameworks; this guide supplies the ModelArk-specific facts to feed into them. The Chinese agent-framework landscape (Coze/扣子, Dify and the rest) is owned by [china_ai_agent_frameworks.md](china_ai_agent_frameworks.md).

---

## 11. The Regulated-Institution Angle

This is a bank's assessment, not a vendor's. The structure is: what a regulated institution must work through **before** using a platform of this class, with the **jurisdictional split made explicit**, because the answer is different for an institution with a mainland entity than for one without.

### 11.1 First question: which jurisdiction's rules are you actually applying?

| Regime | Applies to | Instrument(s) to check | Verified here? |
|---|---|---|---|
| **Mainland China** | Any bank entity, branch or licensed activity in the PRC; any service offered to PRC users | 《生成式人工智能服务管理暂行办法》 (Interim Measures for the Management of Generative AI Services) — filing/备案 and 登记 regime, administered by CAC and local cyberspace authorities; PRC PIPL, Data Security Law and Cybersecurity Law; PRC banking/outsourcing rules from the regulator of record | ⚠️ **Partly verified:** the 备案/登记 regime and its mechanics were verified from the CAC's own announcement (§7.2). PRC banking-sector outsourcing and model-risk instruments were **not** verified in this research |
| **Singapore** | Banks licensed by MAS, including foreign bank branches | MAS instruments on technology risk, outsourcing and AI risk management; the FEAT principles for AI use; MAS expectations on third-party concentration | ❓ **Not verified in this research.** Named as the regimes to check, not asserted as current requirements |
| **Hong Kong SAR** | Authorised institutions supervised by the HKMA | HKMA supervisory guidance on the use of generative AI, plus its outsourcing and technology-risk expectations | ❓ **Not verified in this research** |
| **European Union** | EU-established institutions; EU-located counterparties or data subjects | The **EU AI Act** (Regulation (EU) 2024/1689) for AI-system obligations, and **DORA** (Regulation (EU) 2022/2554) for ICT third-party and operational-resilience requirements, including the register of information on ICT third parties | ❓ Existence widely established; **specific applicability was not verified here** |
| **UK / US / other** | Institution-specific | PRA/FCA outsourcing and operational-resilience rules; US interagency third-party risk guidance and SR model-risk expectations | ❓ Not verified here |

⚠️ **The point of this table is to be explicit about what it is not.** This guide verified the **Chinese** filing regime from the regulator's own site. It did **not** verify the current state of the MAS, HKMA, EU or UK instruments. **Do not quote this table as legal advice or as a compliance checklist** — it is a map of where to look, with the PRC cell filled in.

### 11.2 The six workstreams a bank must complete

**1. Outsourcing and third-party risk approval.** A model-inference platform is a material third-party service. Expect: a formal outsourcing assessment, business-continuity and exit plans, sub-outsourcing disclosure, audit and access rights, and a concentration-risk assessment. 🔗 The outsourcing and operational-risk machinery is condensed in [enterprise_risk_management_guide.md](../../banking/enterprise_risk_management_guide.md) — use it rather than rebuilding it.

**2. Data governance and residency.** Three questions, in order: *what data leaves the bank* (prompt and document content, and any PII or customer identifiers in them); *where it is processed* (region selection — noting the documented possibility of cross-region routing, §7.4); and *what the vendor may do with it* (the executed terms, not the blog, §7.3). 🔗 The supervisory framing lives in [ai_genai_banking_compliance_guide.md](../../banking/ai_genai_banking_compliance_guide.md) and [implementing-responsible-ai.md](implementing-responsible-ai.md).

**3. Model risk management.** A model you did not build, trained on data you cannot inspect, updated on a vendor's schedule, is a model-risk problem. Expect: inventory and ownership, validation independent of the deployment team, performance monitoring in production, change management when the vendor ships a new dated version, and a documented fallback. The dated-suffix model IDs (§3.5) are both a help and a hazard: they make version pinning possible, but **an unpinned integration is an unmanaged model change**. 🔗 Method: [ai_governance_framework_guide.md](ai_governance_framework_guide.md), [llm_evaluation_frameworks_guide.md](llm_evaluation_frameworks_guide.md) and [ai_governance_bias_redteaming_guide.md](ai_governance_bias_redteaming_guide.md).

**4. Security review.** The pass/fail items for this platform specifically: private network connectivity (❓ unverified), customer-managed keys (❓ not found), SIEM-exportable audit logs (❓ unverified), tenant isolation scope (⚠️ vendor claim only), and the certification scope statements plus the SOC 2 Type II report (✅ certificates are listed; **the reports must be obtained under NDA**). See §8.

**5. Content, conduct and consumer-protection review.** For a customer-facing Chinese-language service: content moderation is both a regulatory requirement in the PRC and a conduct risk everywhere. **Do not rely on platform guardrails as the control** (§7.5) — build the bank's own input/output filtering, escalation and human-review path.

**6. Concentration risk and exit.** The realistic question is not "can we switch vendor" but "can we switch vendor in a quarter without losing the product". Required artefacts: a documented exit plan, a data-egress path, evidence that prompts and evaluation sets are portable, and a **second provider that is actually exercised** rather than contracted. Note for this platform specifically: the model IDs are vendor-specific, so portability means **re-baselining prompts and re-running evaluation**, not swapping a configuration string. 🔗 Vendor-lock-in analysis: [enterprise_ai_platforms_guide.md](enterprise_ai_platforms_guide.md).

### 11.3 The jurisdictional split, stated plainly

| Situation | What it means |
|---|---|
| **The institution has a mainland-China entity and serves PRC users** | The CAC 备案/登记 regime applies directly. The bank must ensure the model service it uses is filed, that its own application is registered where required, and that it **publishes the model name and 备案号** in the product. PRC data-protection and banking-outsourcing rules apply. **This is a fundamentally different proposition from the international variant** |
| **The institution has no mainland entity and serves non-PRC customers** | The PRC regime does not apply to it. It would engage the **BytePlus** entity, Singapore/EU hosting and the Singapore/EU compliance story. The relevant constraints are then MAS or HKMA or EU rules — not CAC |
| **The institution has a mainland entity but wants to serve customers outside the PRC using a Chinese platform** | ⚠️ **The hardest case.** The mainland platform's data is in mainland regions under PRC law; a non-PRC regulator may object to the data path, and PRC law may constrain the egress. This configuration requires both jurisdictions' counsel and is where most programmes fail |
| **The institution wants EU residency for EU customer data** | `eu-west-1` (Dublin) exists, but the cross-region routing statement (§7.4), the lack of verified private networking and the unverified audit posture must all be resolved first. **EU AI Act and DORA obligations would sit on the bank, not on the vendor** |

### 11.4 The honest supervisory position

❓ **Not verified in this research:** whether any named regulator (CAC, MAS, HKMA, the EU authorities) has published a position **specifically on Chinese-origin model platforms**, or specifically on 火山方舟/ModelArk. No such guidance was located. **Absence of a specific prohibition is not authorisation** — it means the institution must rely on its own outsourcing, data-governance and model-risk frameworks, and should expect the supervisor to ask about the data path, the concentration risk and the exit plan rather than about the vendor's brand.

### 11.5 The regulated-institution verdict

| Question | Answer |
|---|---|
| Can a regulated institution use this platform at all? | ✅ **Yes, in principle** — subject to the six workstreams above; nothing found makes it categorically prohibited |
| Which variant would a bank use? | Depends entirely on where its entity and its customers are — see §11.3 |
| Is the compliance evidence complete? | ❌ **No.** The 豆包/火山方舟 filing entry, private networking, audit export, BYOK and the executed data terms were all unresolved in this research |
| Is concentration risk material? | ⚠️ **Yes** — a single-vendor stack across models, inference, agents, retrieval and tooling |
| Is cross-region processing a live data-residency issue? | ⚠️ **Yes** — vendor-documented as possible |
| Would this pass a first-round bank review as-is? | ~ **Not on the retrieved evidence.** A qualified "maybe" requires the open items in §14 to be closed by the vendor |

---

## 12. The Cymbal Bank Worked Example

⚠️ **This entire section is fiction, used as an analytical device.** Cymbal Bank is a fictional institution. **All figures — volumes, costs, timelines, headcount — are illustrative and invented for the purpose of showing method.** They are *not* quotes, *not* market data and *not* derived from any vendor's published price list except where a published unit price is explicitly cited and dated. Do not lift these numbers into a real business case.

### 12.1 The scenario

**Cymbal Bank** is a fictional mid-sized bank with a Singapore-domiciled parent and a **small mainland-China presence** (a representative office and a growing cross-border corporate-banking desk). It runs a Western-cloud estate, including an OpenAI-style assistant built on a hyperscaler model platform, with a well-governed prompt/version pipeline.

**The proposed use case:** a **Chinese-language document-processing and customer-enquiry assistant** for the cross-border corporate desk — reading trade-finance documents (bills of lading, invoices, letters of credit) with Chinese-language content, extracting fields, drafting Chinese-language responses, and answering desk staff questions from an internal policy knowledge base.

**The question:** should this workload go to a Chinese model platform (ModelArk / 火山方舟) or extend the Western platform the bank already uses?

### 12.2 Step 1 — Requirements, stated as pass/fail

| Requirement | Type | Test |
|---|---|---|
| Chinese-language document understanding at production accuracy | Functional | ≥ 95% field-extraction F1 on a 500-document golden set |
| Chinese-language generation quality acceptable to desk staff | Functional | Blind preference test against the incumbent |
| Data residency: customer documents must not leave the bank's approved jurisdiction | **Hard constraint** | Contractual + technical confirmation of processing location |
| No training on Cymbal Bank inputs | **Hard constraint** | Executed contract term, not a blog statement |
| Private network path from the bank's VPC to the model endpoint | **Hard constraint** (bank policy) | Private connectivity available and demonstrable |
| Audit logs exportable to the bank's SIEM | **Hard constraint** | Demonstrable integration |
| Model version pinning | Hard constraint | Exact dated model ID or endpoint pinned in config |
| Cost per 1,000 documents below the incumbent | Commercial | Measured, not modelled, at conclusion of pilot |
| Exit within one quarter | Hard constraint | Documented plan, exercised |

⚠️ **Note that only three of these are about model quality.** That is the shape of every real platform decision in a regulated institution.

### 12.3 Step 2 — Capability fit

| Requirement | ModelArk finding | Verdict |
|---|---|---|
| Chinese-language document understanding | ✅ **Document understanding** page exists; multimodal vision and document modalities documented; the model family is Chinese-language-first | **Pass on documentation; verify by test** |
| Field extraction | ✅ Structured output (documented, **labelled beta**) and function calling | Pass with beta risk |
| Chinese-language generation | ✅ Flagship family is Chinese-language-oriented | Pass on documentation |
| Policy knowledge base | ✅ Managed knowledge base with retrieval API, slice management and Q&A exists — the specific feature the workload needs | **Pass on documentation** |
| Multimodal document input (scanned images) | ✅ Image + document understanding documented | Pass |
| Model-quality parity with the incumbent | ❓ **No independent benchmark exists** (§9.3) | **Must be tested by the bank** |

**Illustrative pilot design:** 500 golden documents, 1,000 desk enquiries, three models (a flagship, a mid-tier and a small variant), run against the bank's existing assistant as the control, with the bank's own evaluation harness. 🔗 Use [llm_evaluation_frameworks_guide.md](llm_evaluation_frameworks_guide.md); do not invent a new methodology.

### 12.4 Step 3 — Data-residency and compliance analysis

| Question | Finding | Outcome |
|---|---|---|
| Which region would host this workload? | The bank's parent is Singapore; PRC entity is small. Options: `ap-southeast-1` (Johor, MY) via BytePlus, or a mainland region via 火山方舟 | **Decision point** |
| Does the PRC filing regime apply? | Only if the workload is offered to PRC users via a PRC entity. The desk serves cross-border clients from Singapore | **Likely not applicable if served from Singapore** — must be confirmed with counsel |
| Would a mainland deployment trigger 备案/登记 obligations for the bank's application? | ✅ Yes, if the bank were the service provider to PRC users on a mainland platform | **Materially increases the compliance workload** |
| Can the vendor guarantee the data stays in the chosen region? | ❌ **No** — the vendor documents possible cross-region routing for capacity scheduling (§7.4) | **BLOCKER until resolved in writing** |
| Does the vendor train on inputs? | ❌ **Not established** — blog statements only; terms not read (§7.3) | **BLOCKER until contractual** |
| Is the model service itself filed in the PRC? | ❓ Not verified (§7.2) | Only relevant to a mainland deployment |

**Illustrative conclusion:** the workload is **Singapore-served**, so the PRC regime is probably not engaged and the **BytePlus international variant** is the relevant product — **not** 火山方舟. That single distinction changes the contracting entity, the hosting region, the legal regime and the certification set. **This is the naming-gate finding of §1 paying for itself in practice.**

### 12.5 Step 4 — Security and outsourcing review

| Item | Finding | Action |
|---|---|---|
| Private connectivity | ❓ Not verified | **Ask the vendor; treat as a gate** |
| SIEM-exportable audit logs | ❓ Not verified | **Ask; treat as a gate** |
| Customer-managed keys | ❓ Not found | **Ask; higher bar than the bank's usual standard** |
| Tenant isolation | ⚠️ "Logical isolation" claimed | Ask for the isolation boundary in writing |
| Certifications | ✅ ISO 27001/27017/27018/27701, SOC 1/2/3, CSA STAR, MTCS L3, DPTM, APEC CBPR/PRP on the BytePlus trust centre | **Request the SOC 2 Type II report and scope statements under NDA** |
| Sub-outsourcing disclosure | ❓ Not researched | Standard outsourcing questionnaire item |
| Exit and portability | ⚠️ Vendor-specific model IDs | Requires re-baselining, not reconfiguration |

**Illustrative outsourcing classification:** *material* — the workload touches customer documents and feeds a customer-facing process. That triggers the full approval path (committee, BCP, exit plan, annual review), not a light-touch vendor onboarding.

### 12.6 Step 5 — Cost comparison (ILLUSTRATIVE AND INVENTED)

⚠️ **These numbers are made up for illustration.** Only the two unit prices marked ✅ are real published prices, dated 2026-09-16. Everything else — volumes, the incumbent's price, the FX rate, the totals — is fiction.

| Line item | Incumbent (Western platform) | ModelArk | Basis |
|---|---|---|---|
| Assumed annual document volume | 120,000 documents | 120,000 documents | Illustrative |
| Assumed tokens per document (input+output) | 6,000 tokens (80/20 in/out split) | 6,000 tokens | Illustrative |
| Assumed annual tokens | ~720 M | ~720 M | Illustrative |
| Input unit price | $2.50 / M tokens (invented) | ✅ **$0.50 / M tokens** (`dola-seed-2-1-turbo`, published 2026-09-16) | First real; second invented |
| Output unit price | $10.00 / M tokens (invented) | ✅ **$2.50 / M tokens** (same source) | First real; second invented |
| Illustrative annual inference cost | $2,880 | **$576** | Arithmetic on the above |
| **Cache effect (illustrative)** | — | Assume 60% of input is a stable policy prefix → cache-hit at **$0.10/M** instead of $0.50/M | ✅ Cache price is real; the 60% assumption is invented |
| Illustrative annual inference cost after caching | $2,880 | **≈ $490** | Arithmetic |
| **Cost of the compliance gap** | $0 (already approved) | **Material, unbudgeted, and possibly disqualifying** | See §12.5 |
| **Cost of dual-running during exit-capability build** | — | ≈ 30% of run cost for 2 quarters | Illustrative |

⚠️ **The honest conclusion of this table is not "ModelArk is 5× cheaper".** It is that the **modelled inference saving is small in absolute terms at this volume** — a few thousand dollars a year — while the **compliance, integration and dual-run costs are orders of magnitude larger and harder to quantify.** For a volume this size, **unit price is not the deciding factor, and a business case built on per-token price would be the wrong business case.** The case has to rest on capability (does it read Chinese trade documents better?) and on a market requirement (does the desk need it?).

### 12.7 Step 6 — Integration and migration effort

| Workstream | Effort | Note |
|---|---|---|
| Wire-layer migration | **Low** (days) | Base URL, key, model ID — the wire shape is OpenAI-compatible (§5.1) |
| Prompt re-baselining | **Medium** (weeks) | Behaviour differs; the bank's prompt library needs rework and re-review |
| Evaluation re-run | **Medium** (weeks) | Golden sets are reusable if the harness is model-agnostic |
| Knowledge base migration | **Medium** | Ingestion, chunking and retrieval evaluation — 🔗 `technology/ai_llm/rag/` |
| Observed logging / SIEM | **Unknown** | Depends on whether export exists (§8.4) |
| Governance artefacts | **High** (months) | Outsourcing approval, model-risk registration, monitoring, exit plan |
| **Total** | **Governance dominates** | The engineering is fast; the assurance is slow |

### 12.8 Step 7 — Exit and portability

**Exit plan sketch (illustrative):** keep the model calls behind the bank's own abstraction layer; store prompts and golden evaluation sets in the bank's repository rather than in the vendor console; retain a **contracted and periodically exercised** second provider; require a data-egress commitment; and run a **quarterly "switch test"** on a non-production copy of the workload. ⚠️ **Note the asymmetry:** because Ark model IDs are vendor-specific, exit means *re-baselining and re-evaluating*, not swapping a string — so the exit plan is a **capability**, not a document.

### 12.9 The recommendation

**Recommendation (illustrative):** **run a bounded, production-representative pilot on the BytePlus international variant in `ap-southeast-1`, in parallel with the incumbent, and do not commit until the five gate items in §12.5 are answered in writing.**

Rationale: the capability case for a Chinese-language document workload is **credible but unproven**, and would be established by the pilot, not by this guide. The commercial case is **weak at this volume** — the modelled saving is trivial relative to the assurance cost. The compliance case is **unresolved**: the residency guarantee, the training-on-inputs term, the audit-export path and the certification scope are all open. A mainland 火山方舟 deployment is **not recommended** for this workload, because it engages the PRC filing regime for no benefit the desk actually needs.

**What the bank CANNOT establish, and must say so in the paper:** whether the vendor trains on inputs; whether the vendor can contractually guarantee single-region processing; whether the service is filed in the PRC register; whether private networking and SIEM export exist; whether the performance claims in §9 survive the bank's own load test; and whether the accuracy claims survive the bank's own golden set.

---

## 13. The Claims Audit

Every material claim in this guide, with its source, the source's quality, and the date the claim is tied to. **Model names, prices and capabilities go stale fastest** — the date column is the expiry warning.

| # | Claim | Verdict | Source | Source quality | Dated |
|---|---|---|---|---|---|
| 1 | The international platform is branded **ModelArk**; the mainland platform is **火山方舟 / VolcanoArk** | ✅ Verified | `docs.byteplus.com/en/docs/ModelArk/*`; `volcengine.com/product/ark`; Baidu Baike foreign-name field | Vendor primary + secondary | 2026-09-16 |
| 2 | BytePlus is the international arm; entity is **BytePlus Pte Ltd.** (Singapore) | ✅ Verified | [byteplus.com/en/trust-center](https://www.byteplus.com/en/trust-center), page footer | Vendor primary | 2026-09-16 |
| 3 | International base URLs are `ark.ap-southeast.bytepluses.com` (AP, Johor) and `ark.eu-west.bytepluses.com` (EU, Dublin) | ✅ Verified | [Region availability](https://docs.byteplus.com/en/docs/ModelArk/2191806) | Vendor primary | 2026-09-16 |
| 4 | Mainland console/API is region-scoped under `ark.volcengine.com/region:cn-beijing` | ✅ Verified | Console URL pattern in vendor pages | Vendor primary | 2026-09-16 |
| 5 | The mainland API endpoint pattern `ark.cn-beijing.volces.com` | ❓ Not directly seen in this research | — | — | — |
| 6 | 火山方舟 announced **2023-06-28** at the V-Tech summit, invite beta, 7 integrated models | ❓ Reported | Baidu Baike citing 界面新闻 2023-06-28 | Secondary | 2023-06-28 |
| 7 | Doubao family opened externally **2024-05-15** at FORCE, 9 members, "厘时代" pricing | ❓ Reported | Zhihu FORCE write-up | Secondary, promotional-adjacent | 2024-05-15 |
| 8 | Platform opened to individual developers **2024-06** | ❓ Reported | Baike citing 百家号 | Secondary | 2024-06 |
| 9 | Doubao 1.6 + Seedance 1.0 pro launched at FORCE; **input-length tiered pricing**; ¥0.80/¥8 per M in the 0–32K band; 63% below Doubao 1.5; Seedance ≈¥3.67 per 5s 1080p | ✅ Event / ⚠️ percentage | [36Kr reprint of 界面新闻](https://m.36kr.com/p/3333345351248136) | Third-party business press (detailed) | 2025-06-12 |
| 10 | Doubao daily tokens **>16.4 trillion**, **137×** the May-2024 level | ⚠️ Vendor claim | Same article, quoting 火山引擎 | Vendor-reported via press | 2025-06-12 |
| 11 | Doubao **46.4%** #1 share of China public-cloud LLM (IDC) | ❓ Unverified | Same article | Second-hand statistic; IDC report not retrieved | 2025-06-12 |
| 12 | Doubao-Seed-Code released 2025-11-11; claimed 62.7% below industry average cost; ¥9.9 first month Coding Plan | ⚠️ Vendor claim | Baike citing 百家号 2025-11-11 | Secondary; vendor claim | 2025-11-11 |
| 13 | 豆包助手 API on 火山方舟 with 联网搜索 etc. | ❓ Reported | Baike citing 百家号 2025-12-18 | Secondary | 2025-12-18 |
| 14 | Current BytePlus catalogue: **Dola Seed 2.1 turbo** (2026-07-13), **Dola Seed 2.0 pro/lite/mini** (2026-03-24 / 03-06 / 02-14), **Seedream 5.0 Pro/Lite**, **Seedance 2.5**, Seed Speech TTS/ASR 2.0, Omnihuman, DreamActor | ✅ Verified (as published) | [ai.byteplus.com/en/model](https://ai.byteplus.com/en/model) | Vendor primary | 2026-09-16 |
| 15 | **Dola** is the international brand for the Seed model family | ✅ Verified (usage) / ⚠️ rename-only-inferred | BytePlus blog + model page + docs model IDs | Vendor primary | 2026-03-24 / 2026-09-16 |
| 16 | Third-party **DeepSeek** models are hosted on the platform (`deepseek-v4-flash-260425`, `-pro-260425`, `-ga-260731`) | ✅ Verified | [ModelArk pricing](https://docs.byteplus.com/en/docs/ModelArk/1544106) | Vendor primary | 2026-09-16 |
| 17 | Model IDs carry **date suffixes** and qualifiers (`-ga`, `-turbo`, `-mini`, `-lite`, `-pro`, `-fast`, `-thinking`) | ✅ Verified | Docs + pricing tables | Vendor primary | 2026-09-16 |
| 18 | API is **Chat Completions-shaped** (`POST /chat/completions`, `Authorization: Bearer`) and also offers a **Responses API** | ✅ Verified | Region availability; Overview; Responses API doc tree | Vendor primary | 2026-09-16 |
| 19 | SDKs/tabs: **Python (`arkruntime`), cURL, Go, Java, OpenAI SDK**; an "official Ark SDK" exists | ✅ Verified | Quick start sample; "Migrate to the official Ark SDK" | Vendor primary | 2026-09-16 |
| 20 | **Rate limits are not guaranteed** ("theoretical maximum values") | ✅ Verified | [Model list](https://docs.byteplus.com/en/docs/ModelArk/1330310) | Vendor primary (self-limiting admission) | 2026-09-16 |
| 21 | **Region isolation**: per-region endpoints, keys, activation; endpoints callable only via their region's URL | ✅ Verified | Region availability | Vendor primary | 2026-09-16 |
| 22 | **Cross-region routing is possible** ("some requests may be routed to inference resources in other regions") | ✅ Verified | Region availability | Vendor primary | 2026-09-16 |
| 23 | Pricing columns include **cache-storage per M-token per hour** and a discounted **cache-hit input** price | ✅ Verified | ModelArk pricing | Vendor primary | 2026-09-16 |
| 24 | Published prices: `deepseek-v4-flash-260425` $0.44/$1.32/$0.014; `deepseek-v4-pro-260425` $1.32/$3.96/$0.044; `dola-seed-2-1-turbo` $0.50 in / $0.10 cache-hit / $2.50 out / $0.0083 cache-storage | ✅ Verified (published) | ModelArk pricing | Vendor primary | 2026-09-16 (adjusted 2026-08-21 / 08-28) |
| 25 | Documented capabilities: inference endpoints, Flex, **Model Unit**, batch inference, **context cache**, prefill-based response, Files API, fine-tuning with dataset format, **managed knowledge base**, function calling, **MCP**, **Managed Agents** (sandbox/memory/multi-agent/tool-permission policy), **PromptPilot**, App Lab | ✅ Verified (exist) | ModelArk doc tree | Vendor primary (existence only) | 2026-09-16 |
| 26 | Fine-tuning **techniques** offered (LoRA vs full SFT vs preference methods) | ❓ Not verified | — | — | — |
| 27 | 256K-class context on flagship models | ⚠️ Partly verified | Pricing band `[0, 256]` K; 2025 press | Vendor pricing + secondary | 2026-09-16 |
| 28 | "High-performance" knowledge-base retrieval; Toutiao/Douyin-grade plugins; web-search plugin | ⚠️ Vendor claim | 火山方舟 product page | Vendor marketing | 2026-09-16 |
| 29 | "Abundant GPU pool", "minutes to a thousand cards", "instant availability" | ⚠️ Vendor claim | 火山方舟 product page | Vendor marketing | 2026-09-16 |
| 30 | No independent latency/throughput/availability benchmark exists | ✅ Negative finding | — | — | 2026-09-16 |
| 31 | Certifications (ISO 27001/27017/27018/27701, ISO 9001/20000/22301, SOC 1/2/3, CSA STAR, PCI DSS, MTCS L3, DPTM, APEC CBPR/PRP; PDPA + GDPR alignment) belong to **BytePlus** | ✅ Verified | [Trust center](https://www.byteplus.com/en/trust-center) | Vendor primary | 2026-09-16 |
| 32 | Mainland platform claims **SOC 2 + ISO-series** assessment | ⚠️ Vendor claim (secondary) | Baike summarising product page | Secondary | 2026-09-16 |
| 33 | Mainland security claims: mutual-trust computing, 数据落盘即加密, 保距加密+差分隐私, 安全防护栏, 智能脱敏, 访问可控可审计 | ⚠️ Vendor claim | 火山方舟 product page / Baike | Vendor marketing | 2026-09-16 |
| 34 | Vendor blog: data processed on instruction; no access without permission; logical isolation | ⚠️ Vendor claim (not contractual) | [BytePlus blog 2026-03-24](https://www.byteplus.com/en/blog/dola-seed-2-0-pro) | Vendor statement | 2026-03-24 |
| 35 | Contracts/ToS do **not** train on inputs, or do | ❌ **Not established** | ToS/Privacy not read | Gap | — |
| 36 | **PRC 备案 regime exists** under the Interim Measures, administered by CAC + local authorities; API-calling applications go through **登记**; live apps must publish **model name + 备案号** | ✅ Verified | [CAC 2025-09-10 公告](https://www.cac.gov.cn/2025-09/10/c_1759222982377536.htm) | **Regulator primary** | 2025-08-31 data |
| 37 | Cumulative **538 备案 / 263 登记** as of 2025-08-31; 99 备案 + 27 登记 added Jul–Aug 2025 | ✅ Verified | Same CAC announcement | **Regulator primary** | 2025-08-31 |
| 38 | 豆包 / 火山方舟 holds a specific 备案 number | ❓ **Not verified** | Register list not parsed | Gap | — |
| 39 | Third-party totals of 611 / 748 / 796 filed models | ❌ Rejected as authoritative | CSDN / Sohu / Tencent dev blog | Low-quality secondary, mutually inconsistent | 2026 |
| 40 | Interim Measures effective date (reported August 2023) | ❓ Not primary-verified | — | — | — |
| 41 | Competitors 百炼 / 混元 / 千帆 exist as described | ✅ Verified | aliyun.com/product/bailian; cloud.tencent.com/product/tclm; cloud.baidu.com/product-s/qianfan_home | Vendor primary | 2026-09-16 |
| 42 | Tencent Hunyuan capabilities migrating to "TokenHub"; original platform closing to new onboarding | ❓ Partially read | cloud.tencent.com document page | Vendor primary, incomplete | 2026-09-16 |
| 43 | Private networking / VPC endpoint, BYOK, SIEM audit export for the model service | ❓ Not verified | — | Gap | — |
| 44 | "Session trace-free" / 不留痕 claim | ❓ **Not verified; not asserted** | — | Gap | — |
| 45 | CNCF Volcano (volcano.sh) is a different product | ✅ Verified (well-established) | — | General knowledge | — |

---

## 14. What Could Not Be Verified

Honest list, in rough order of how much it matters. **Every item here is a question to put to the vendor, not a finding against them.**

1. **The 豆包 / 火山方舟 entry in the CAC 备案 register, and its filing number.** The regime is verified from the regulator; the specific entry is not. Highest-value open item.
2. **The contractual data terms** — whether BytePlus/Volcengine train on customer inputs, what the retention period is, what metadata is retained, and what the deletion SLA is. Blog statements are not contracts; the ToS and privacy policy were not read line by line in this research.
3. **Whether a customer can contractually or technically prevent cross-region processing.** The vendor documents cross-region routing as possible and does not document an opt-out.
4. **Private/dedicated deployment of model inference** — VPC endpoint, private link, single-tenancy, IP allow-listing. Nothing was found either way.
5. **Customer-managed keys (BYOK/HSM/KMS)** and key rotation or scoping policy.
6. **Audit and observability export** — SIEM integration, API call logs, admin-action trails, retention configuration.
7. **The fine-tuning method matrix** (LoRA vs full-parameter vs preference optimisation), eligible base models, and tuning prices.
8. **The complete price list** — Model Unit/reserved capacity, batch discounts, knowledge-base and vector-storage pricing, image/video generation pricing, Coding Plan price, free-tier entitlements.
9. **Model-level context windows, tokeniser identity and truncation semantics** for the 2026 catalogue.
10. **Error taxonomy and retry/idempotency semantics.**
11. **Any independent benchmark** of latency, throughput, availability or model quality on this platform. None was found. All performance numbers in circulation trace back to the vendor.
12. **The mainland 火山方舟 documentation and pricing** — several `docs.volcengine.com` and `volcengine.com` pages returned **HTTP 500 / were not retrievable from this host**, and the mainland pricing page is JS-rendered. Mainland detail in this guide therefore rests on the product page, Baidu Baike (a secondary, vendor-derived source) and Chinese business press.
13. **Whether "ModelArk" was ever a rename**, and what the product was called in English before
2026. No renaming announcement was found.
14. **The Interim Measures' effective date**, from a primary CAC page.
15. **The current state of MAS / HKMA / EU / UK supervisory instruments** as they apply to a platform of this class, and whether any regulator has published guidance specifically on Chinese-origin model platforms. Not verified here.
16. **Whether ByteDance's own open-weight models are served on ModelArk.**
17. **Enterprise contracting options** — committed-spend discounts, negotiated terms, SLAs, indemnities.
18. **The "session trace-free" / 不留痕 claim** — no primary page containing it was located; this guide therefore does not assert it.
19. **The full list of third-party models hosted** (the pricing and model-list pages retrieved show DeepSeek and first-party models; the catalogue is larger and changes monthly).
20. **Tencent Hunyuan's product migration status** (TokenHub), which affects any competitive comparison built on it.

⚠️ **A structural note on source quality in this domain.** A large share of the searchable material about this platform is reseller marketing, API-aggregator SEO pages and AI-directory listings that recycle vendor numbers without attribution. Those sources were **not** treated as evidence here. Where a fact could not be established from a vendor primary page, a regulator page or credible business press, it is listed above as unverified rather than filled in.

---

## 15. Glossary

| Term | Chinese | Meaning |
|---|---|---|
| Volcano Engine / Volcengine | 火山引擎 | ByteDance's cloud platform (the counterweight to AWS or Alibaba Cloud) |
| Volcano Ark | 火山方舟 | The model-as-a-service platform inside Volcengine — the **mainland** product name |
| ModelArk | *(English brand only)* | The **international** name of the same platform, sold by BytePlus |
| BytePlus | *(English brand)* | ByteDance's international enterprise technology arm; **BytePlus Pte Ltd.** is the contracting entity |
| Doubao | 豆包 | ByteDance's model family brand — **and** the consumer AI assistant app. Not the platform |
| Dola Seed | *(English brand)* | The international-facing branding of the Seed model family on BytePlus |
| Seed | *(English brand)* | ByteDance's research-led model family; published at `seed.bytedance.com` |
| Seedream / Dreamina Seedream | 即梦 / Seedream | Image generation models |
| Seedance / Dreamina Seedance | *(brand)* | Video generation models |
| Coze | 扣子 | ByteDance's no-code agent/app development platform (separate product) |
| MaaS | — | Model-as-a-Service: models delivered as a metered API rather than as software |
| Endpoint | 接入点 / 推理接入点 | A customer-created, region-scoped resource that fronts a model for inference |
| Model ID | 模型 ID | The dated identifier of a specific model version (e.g. `-260425`) |
| Model Unit | 模型单元 | Reserved/committed capacity construct |
| Context cache | 上下文缓存 | Caching of prompt prefixes; priced lower on hit, with an hourly storage charge |
| 备案 | 备案 | **Filing** of a generative-AI service with the CAC — the PRC registration regime for model providers |
| 登记 | 登记 | **Registration** for an application or feature that calls an already-filed model (administered locally) |
| Interim Measures | 《生成式人工智能服务管理暂行办法》 | Interim Measures for the Management of Generative AI Services — the operative PRC instrument for 备案 |
| CAC | 国家网信办 / 中央网信办 | Cyberspace Administration of China — the regulator administering the filing regime |
| Bailian | 百炼 | Alibaba Cloud's model service platform (Model Studio) — a Chinese competitor |
| Hunyuan | 混元 | Tencent's large-model family — a Chinese competitor |
| Qianfan | 千帆 | Baidu AI Cloud's model platform — a Chinese competitor |
| ERNIE | 文心 | Baidu's model family |
| Qwen | 通义千问 | Alibaba's model family |
| TRAE | *(brand)* | ByteDance's AI coding product |
| 厘时代 | 厘时代 | "Per-milli era" — the sub-cent pricing framing used at the May 2024 launch |
| 互信计算 | 互信计算 | "Mutual-trust computing" — the mainland security-architecture term for protecting model-provider IP |
| 安全防护栏 | 安全防护栏 | "Safety guardrails" — the vendor's content-risk identification feature |
| 智能脱敏 | 智能脱敏 | "Intelligent de-identification/masking" of sensitive content |
| 联网插件 | 联网插件 | The web-search (internet-connected) plugin |
| PDPA / GDPR | — | Singapore's Personal Data Protection Act / the EU's General Data Protection Regulation |

---

## 16. Cross-References and Further Reading

**Within this repository — sibling guides that own their own disciplines (do not duplicate them):**

| Guide | What it owns |
|---|---|
| [enterprise_ai_platforms_guide.md](enterprise_ai_platforms_guide.md) | The Western-platform comparison (Bedrock/Azure AI/Databricks/SageMaker/Vertex AI), the platform-evaluation criteria, the governance frameworks and the **vendor-lock-in analysis** |
| [china_ai_agent_frameworks.md](china_ai_agent_frameworks.md) | The Chinese agent-framework landscape — Coze/扣子, Dify and the rest |
| [fine_tuning_frameworks_comparison_guide.md](fine_tuning_frameworks_comparison_guide.md) | Fine-tuning method selection |
| [llm_evaluation_frameworks_guide.md](llm_evaluation_frameworks_guide.md) | Evaluation methodology and model-change validation |
| [ai_governance_framework_guide.md](ai_governance_framework_guide.md) | AI governance frameworks and model-risk expectations |
| [implementing-responsible-ai.md](implementing-responsible-ai.md) | Responsible-AI implementation |
| [ai_governance_bias_redteaming_guide.md](ai_governance_bias_redteaming_guide.md) | Bias testing and red-teaming |
| [ollama_xinference_localai_guide.md](ollama_xinference_localai_guide.md) | Self-hosted / on-premises alternatives |
| `technology/ai_llm/rag/` | RAG architecture, chunking, retrieval evaluation and vector databases |
| [ai_genai_banking_compliance_guide.md](../../banking/ai_genai_banking_compliance_guide.md) | The bank-compliance and supervisory angle for generative AI |
| [enterprise_risk_management_guide.md](../../banking/enterprise_risk_management_guide.md) | Outsourcing, operational risk and third-party risk management |

**Primary and secondary sources used (all accessed 2026-09-16 unless dated otherwise):**

| Source | Type | Quality |
|---|---|---|
| [BytePlus ModelArk — platform overview](https://docs.byteplus.com/en/docs/ModelArk/1099455) | Vendor product documentation | Primary for capability and API shape; **not** independent |
| [BytePlus ModelArk — model list](https://docs.byteplus.com/en/docs/ModelArk/1330310) | Vendor documentation | Primary for catalogue and rate-limit language |
| [BytePlus ModelArk — pricing](https://docs.byteplus.com/en/docs/ModelArk/1544106) | Vendor documentation | Primary for published prices |
| [BytePlus ModelArk — region availability](https://docs.byteplus.com/en/docs/ModelArk/2191806) | Vendor documentation | Primary for endpoints and isolation |
| [BytePlus — product page](https://www.byteplus.com/en/product/modelark) and [AI model page](https://ai.byteplus.com/en/model) | Vendor marketing | Capability claims = marketing; model inventory = useful |
| [BytePlus — trust centre](https://www.byteplus.com/en/trust-center) | Vendor compliance page | Primary for certification **listings**; reports require NDA |
| [BytePlus blog — Dola Seed 2.0 Pro (2026-03-24)](https://www.byteplus.com/en/blog/dola-seed-2-0-pro) | Vendor announcement | Primary for the vendor's own data-protection wording; **marketing** for capability |
| [Volcengine 火山方舟 product page](https://www.volcengine.com/product/ark) | Vendor marketing (JS-heavy; limited extraction) | Marketing claims only |
| [Baidu Baike — 火山方舟](https://baike.baidu.com/item/%E7%81%AB%E5%B1%B1%E6%96%B9%E8%88%9F/63145443) | Encyclopedia, sourced from vendor + press | Secondary; useful for dates and the foreign-name field; vendor-derived |
| [CAC announcement, 2025-09-10](https://www.cac.gov.cn/2025-09/10/c_1759222982377536.htm) | **Regulator primary** | Highest quality in this dossier for the filing regime |
| [36Kr reprint of 界面新闻, 2025-06-12](https://m.36kr.com/p/3333345351248136) | Third-party business press | Good for events and quoted figures; vendor numbers repeated uncritically |
| [Zhihu FORCE conference write-up](https://zhuanlan.zhihu.com/p/698265798) | Community write-up | Secondary; useful for the 2024 launch framing |
| [seed.bytedance.com — models](https://seed.bytedance.com/en/models) | Vendor research site | Primary for the research family; **not** a service commitment |
| [Alibaba Cloud 百炼](https://www.aliyun.com/product/bailian) · [Tencent Hunyuan](https://cloud.tencent.com/product/tclm) · [Baidu Qianfan](https://cloud.baidu.com/product-s/qianfan_home) | Vendor product pages | Used only to verify the competitors' existence and positioning |

**Not reachable from this host (recorded as a limitation):** several `docs.volcengine.com` and `volcengine.com` pages returned server errors, and `byteplus.com/activity/free` and `api.byteplus.com/api-sdk` rendered only navigation. Mainland pricing and the free-tier entitlements therefore could not be verified first-hand.

---

## 17. Closing Summary

The naming question has a clean answer, and it is the most useful thing in this guide: **火山方舟 (VolcanoArk) is the mainland-China product; ModelArk is the international product sold by BytePlus; Doubao/豆包 is the consumer app and the model-family brand; Seed/Dola Seed is the international model branding; Coze/扣子 is a separate agent platform; and CNCF Volcano has nothing to do with any of it.** A reader who internalises only that mapping, and the fact that the two variants sit under **different legal entities, different regions and different regulators**, will avoid the mistake that matters most — treating one brand as one procurement.

On the substance, the platform is real and substantial. The documentation proves a genuine production MaaS surface: region-scoped inference endpoints, batch and Flex modes, context caching with an hourly storage charge, reserved capacity, fine-tuning with a defined dataset format, a managed knowledge base with retrieval and Q&A, function calling and MCP, and a documented multi-agent runtime with sandboxes, persistent memory and tool-permission policy. The API is OpenAI-shaped at the wire level, which makes migration look easy — and the governance work is what makes it hard. The pricing structure is transparent and has been cut repeatedly, with input-length tiers, cache discounts and two documented adjustments in August 2026 alone.

What is not established matters just as much, and a bank should treat this list as the agenda for its first vendor meeting: **the specific CAC filing entry for 豆包/火山方舟; the contractual data terms rather than the blog statements; whether cross-region processing can be contractually excluded; whether private networking, customer-managed keys and SIEM-exportable audit logs exist; the fine-tuning method matrix and prices; and any independent benchmark of latency, throughput or quality at all.** The vendor's own documentation supplies the sharpest warning in the dossier: its published rate limits are explicitly **not guaranteed**.

For a regulated institution, the honest conclusion is a **conditional one**. Nothing found makes this platform impermissible, but nothing found makes it ready either. The capability case — Chinese-language document understanding, native multimodal generation, an integrated agent runtime — is credible and, for a China-facing or Chinese-language workload, potentially hard to match elsewhere. The assurance case is unfinished, and the commercial case is weak at small volumes, where the modelled per-token saving is dwarfed by the cost of closing the governance gaps. The correct posture is therefore a bounded pilot, a load test, a written answer to each open item, and an exit capability that is exercised rather than documented — because the discipline that makes a platform usable by a bank is not the model, and not the price, and not the brand, but **the model service.**
