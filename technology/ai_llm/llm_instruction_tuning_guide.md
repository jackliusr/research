# LLM Instruction Tuning — Comprehensive Guide

A detailed technical guide to **instruction tuning** — the supervised fine-tuning (SFT) stage that teaches a base language model to follow instructions — covering definition, history, datasets, training mechanics, contrasts with RLHF/DPO, PEFT practice, evaluation, production practice, and a worked banking example.

**Author:** Jack Liu Shurui
**Series:** LLM/AI Technical Guides
**Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)

---

## How to Use This Guide — Related Guides

This guide is the **dedicated deep-dive on instruction tuning** (the SFT stage for instruction-following). It deliberately does **not** duplicate the umbrella coverage elsewhere in this series; it expands a ~3-line subsection of the fine-tuning umbrella into full depth and cross-references its siblings:

| Related guide | What it covers | How this guide relates |
|---|---|---|
| `fine_tuning_frameworks_comparison_guide.md` | The fine-tuning **umbrella**: full vs PEFT, LoRA/QLoRA methods, frameworks, RLHF/DPO/ORPO/GRPO, QAT, data prep, infra, eval, when-to-fine-tune | Instruction tuning is §2.3 there ("Instruction Following"); this guide is the full treatment of that stage. **Cross-reference for PEFT methods (§4), alignment methods (§6), data prep (§8), infra (§9), and post-FT evaluation (§10).** |
| `llm_evaluation_frameworks_guide.md` | LLM evaluation frameworks (DeepEval, LangSmith, RAGAS, G-Eval…), LLM-as-judge, metric categories | Cross-reference for the **evaluation tooling** used after instruction tuning (this guide's §7 focuses on the instruction-following benchmarks themselves) |
| `llm_evaluation_vs_validation_guide.md` | Evaluation vs validation distinction, model risk management framing | Cross-reference for how SFT evals fit into model risk / validation governance |
| `autonomous_agents_guide.md` | Agent architectures (ReAct, tool loops); instruction-following as the agent substrate | Cross-reference for why instruction-following quality is the **base capability** agents build on (this guide §1.3) |
| `prompt_injection_guide.md` | Prompt injection and instruction-boundary attacks | Cross-reference for why instruction-tuned models must respect **boundaries between instructions and data** (this guide §8.2) |
| `rag/rag_evaluation_methodology_guide.md` | RAG evaluation methodology | Cross-reference for **task/domain evals** that complement instruction-following benchmarks (§7.4) |
| `../../banking/core_banking_processes_guide.md`, `../../banking/full_stack_banking_guide.md` | Banking domain context | Cross-reference for the worked banking assistant example (§9) |

**Verification note on this guide:** all historical claims (papers, dates, arXiv IDs, dataset sizes) were verified against primary sources (arXiv abstracts, project pages, Hugging Face dataset cards) at the time of writing. Where a number is a common heuristic or could not be independently confirmed, it is explicitly **flagged** as such. Do not treat flagged figures as verified facts.

---

## Table of Contents

1. [Definition: What Instruction Tuning Is](#1-definition-what-instruction-tuning-is)
2. [History: From FLAN and InstructGPT to the 2023 Data Explosion](#2-history-from-flan-and-instructgpt-to-the-2023-data-explosion)
3. [Instruction Datasets: The Canonical Six](#3-instruction-datasets-the-canonical-six)
4. [SFT Mechanics: Templates, Masking, and Training Setup](#4-sft-mechanics-templates-masking-and-training-setup)
5. [SFT vs RLHF vs DPO: The Alignment Stack](#5-sft-vs-rlhf-vs-dpo-the-alignment-stack)
6. [PEFT for Instruction Tuning: LoRA and Friends](#6-peft-for-instruction-tuning-lora-and-friends)
7. [Instruction-Following Evals: IFEval and Beyond](#7-instruction-following-evals-ifeval-and-beyond)
8. [Practice: Curation, Decontamination, Iteration](#8-practice-curation-decontamination-iteration)
9. [Worked Example: Instruction-Tuning a Banking Assistant](#9-worked-example-instruction-tuning-a-banking-assistant)
10. [Summary: Instruction Tuning in One Page](#10-summary-instruction-tuning-in-one-page)
11. [Glossary](#11-glossary)

---

## 1. Definition: What Instruction Tuning Is

### 1.1 The One-Sentence Definition

**Instruction tuning** is the supervised fine-tuning (SFT) of a pre-trained base language model on a dataset of **(instruction, response) pairs** — "summarize this article → <summary>", "translate to French → <translation>", "extract the counterparty from this SWIFT message → <JSON>" — so that the model learns the *behaviour* of responding to a user's request, rather than merely continuing text.

Three terms are used near-interchangeably in the field, with slightly different emphases:

- **Instruction tuning** — the broadest term; emphasises the *data* (tasks described as natural-language instructions). Popularised by Wei et al.'s FLAN paper (2021), which defined it as *"finetuning language models on a collection of tasks described via instructions."*
- **Supervised fine-tuning (SFT)** — the *training mechanism*: standard supervised learning (maximise next-token likelihood of the gold response) on labelled examples. SFT is a general tool; instruction tuning is its most common application. When you fine-tune on instruction–response pairs, you are doing SFT.
- **Chat tuning / assistant fine-tuning** — instruction tuning applied to conversational data (multi-turn dialogues), typically with a chat template and system prompts. Most modern "chat models" (GPT-3.5 era onwards) are instruction-tuned models.

The mental model: **pre-training gives the model knowledge and language; instruction tuning gives it the behavioural interface — how to respond when told what to do.** LIMA's authors put it memorably: alignment (including instruction tuning) mostly teaches a model *the style and format* of assistant responses, while the underlying knowledge lives in pre-training (Zhou et al., 2023 — see §3.5).

**What the data actually looks like** (the canonical JSON shape, before template rendering — see §4.1):

```json
{
  "instruction": "Explain a letter of credit in three sentences.",
  "input": "",
  "output": "A letter of credit is a bank guarantee that the seller will be paid..."
},
{
  "instruction": "Extract the counterparty, amount, currency and value date from this SWIFT message.",
  "input": "{MT103 field content...}",
  "output": "{\"counterparty\": \"ACME GmbH\", \"amount\": 1250000, \"currency\": \"EUR\", \"value_date\": \"2026-09-30\"}"
}
```

Most open datasets use exactly this `instruction` / `input` / `output` shape (Alpaca, Dolly, LIMA, OASST1 all do); `input` is the optional context/task material, `output` the gold response. Everything downstream — chat templates, loss masking, LoRA training — starts from this shape.

### 1.2 The Base vs the Instruction-Tuned Model

The contrast that matters:

| Dimension | Base model (pre-trained only) | Instruction-tuned model |
|---|---|---|
| **Training objective** | Next-token prediction on raw web/corpus text | Next-token prediction on gold **responses to instructions** |
| **Input format** | Raw text; no notion of "user vs assistant" | Chat/instruction format: system prompt, user turn, assistant turn |
| **Behaviour** | Completes text: "Q: What is a swap? A:" → likely continues with plausible-looking text, but may also answer, ramble, or refuse mid-sentence | Treats the user turn as a *request* and produces a *response* to it, with the right format and length |
| **Zero-shot usability** | Poor — needs few-shot examples or heavy prompting; often ignores constraints | Strong — follows instructions zero-shot, including format and length constraints |
| **Typical failure** | Continues the document instead of answering | Follows the *form* but may hallucinate *content* (knowledge still comes from pre-training) |

A concrete illustration: prompt a base model with *"Write a haiku about trade finance. Respond with JSON only."* The base model might output *"Write a haiku about trade finance. Respond with JSON only. A haiku is a Japanese poem of three lines…"* — it continues the prompt because that is what its pre-training objective rewards. The instruction-tuned model outputs the haiku, in JSON, as asked. This is the entire point of the stage.

### 1.3 Why It Matters: The Usability Rationale

Before instruction tuning (pre-2022), using an LLM meant engineering prompts — few-shot exemplars, careful phrasing — to coax a text-completion engine into behaving like an assistant. That made LLMs a specialist tool. Instruction tuning is the step that turned them into a *product*:

1. **It converts "text completion" into "task execution."** The model stops predicting the next plausible sentence and starts solving the stated task.
2. **It makes models usable by non-experts.** End users phrase requests in natural language; no prompt engineering required.
3. **It is the base capability everything else builds on.** Agents (see `autonomous_agents_guide.md`) rely on the model reliably executing tool-call instructions; RAG pipelines rely on it answering from retrieved context *instead of* ignoring the instructions; structured-output systems rely on it obeying format constraints. If instruction-following is weak, every downstream layer degrades.
4. **It is cheap relative to the alternatives.** Instruction tuning (thousands to tens of thousands of examples, often with LoRA on one GPU) is dramatically cheaper than pre-training, and it can recover much of the gap to far larger models. FLAN's headline result: a 137B instruction-tuned model beat a zero-shot 175B GPT-3 on 20 of 25 tasks (Wei et al., 2021).
5. **It is the foundation of the modern alignment pipeline.** InstructGPT showed that SFT on demonstrations is step 1 of RLHF; DPO assumes an SFT model as its reference policy. No SFT, no alignment (§5).

### 1.4 The Definition Table

| Aspect | Description |
|---|---|
| **What it is** | Supervised fine-tuning on (instruction, response) pairs so the model learns to respond to requests |
| **Training objective** | Standard SFT: maximise next-token likelihood of the gold response tokens (typically response-only, see §4.2) |
| **Input data** | Instruction datasets: human-written, machine-generated, or mixed (see §3) |
| **What changes** | Model weights (full FT) or adapters (LoRA/QLoRA, see §6) — knowledge comes from pre-training, behaviour from SFT |
| **What it is not** | Not pre-training (no new knowledge at scale), not prompt engineering (weights change), not RLHF (no reward model / preference optimisation, see §5) |
| **Deliverable** | An instruction-tuned (chat) model: zero-shot instruction-following, format compliance, usable by end users |
| **Typical scale** | ~1k–50k examples, 1–3 epochs (see §4.3 — the range is a heuristic, flagged) |

---
## 2. History: From FLAN and InstructGPT to the 2023 Data Explosion

Instruction tuning has a short, dense history. Two 2021–2022 papers defined the idea (FLAN) and proved it at frontier scale (InstructGPT); 2023 turned it into an industrial process with open datasets and open models.

### 2.1 FLAN — The Paper That Named the Technique (2021)

**Wei et al., "Finetuned Language Models Are Zero-Shot Learners," arXiv 2109.01652 (September 2021).** Google Research. This is the paper that coined the term **instruction tuning**: fine-tuning a language model on a collection of tasks, each *described via a natural-language instruction* rather than a task-specific input–output format.

Key results (verified against the paper):

- FLAN (Fine-tuned LAnguage Net) was a 137B LaMDA-PT model instruction-tuned on **62 datasets** converted into instruction format, covering 12 task clusters (natural language inference, reading comprehension, translation, summarisation, sentiment, etc.).
- Held-out task evaluation: instruction tuning improved zero-shot performance on unseen tasks — FLAN surpassed zero-shot 175B GPT-3 on **20 of 25** evaluated datasets.
- The key insight: training on *many tasks in instruction form* transfers to *unseen tasks* — the model learns the general skill "do what the instruction says" rather than memorising tasks.
- The paper also documented practical design choices still used today: **template ensembling** (each dataset rendered with multiple instruction phrasings), **held-out task clusters** for honest evaluation, and the observation that instruction tuning helps small models too, with diminishing gains at scale.

FLAN's framing is why the whole technique is called "instruction tuning" and not just "task fine-tuning": the *instruction* is the interface.

### 2.2 InstructGPT — The Paper That Proved the Pipeline (2022)

**Ouyang et al., "Training language models to follow instructions with human feedback," arXiv 2203.02155 (March 2022), NeurIPS 2022.** OpenAI. InstructGPT is the direct ancestor of ChatGPT (ChatGPT launched November 2022 on the same recipe) and is the canonical reference for the **alignment pipeline**: SFT → reward model → RLHF.

The InstructGPT recipe, in three steps:

1. **SFT**: fine-tune GPT-3 on ~13k (instruction, demonstration-response) pairs written by labelers. This is the *instruction tuning* step — exactly the subject of this guide. Labelers wrote instructions covering open-ended generation, brainstorming, QA, summarisation, and more, and wrote the desired responses.
2. **Reward model (RM)**: labelers rank several SFT outputs per prompt; a reward model is trained on these comparisons.
3. **RLHF (PPO)**: the SFT model is optimised against the reward model with PPO, with a KL penalty to the SFT policy.

Headline verified results:

- Labelers preferred 175B **InstructGPT outputs over 175B GPT-3 outputs 85% ± 3%** of the time, and **71% ± 4%** over few-shot GPT-3.
- InstructGPT generalised to instructions outside its training distribution (non-English, code summarisation/QA) — the same transfer property FLAN found.
- Known failure modes acknowledged in the paper: still makes simple mistakes, can make up facts, gives hedging answers, fails to detect instructions with false premises.
- InstructGPT was 100× cheaper to train than GPT-3's pre-training (1.3M human-written/ranked tokens vs ~300B+ pre-training tokens), and a 1.3B InstructGPT beat a 175B GPT-3 on the API prompt distribution — **the "small tuned model beats giant base model" result** that underwrote the entire fine-tuning economy.

For this guide the crucial takeaway: **InstructGPT's SFT step (step 1) is instruction tuning proper**; RLHF is the follow-on preference stage (§5).

### 2.3 The Timeline: 2021–2023 Milestones

| Date | Milestone | Significance |
|---|---|---|
| **Sep 2021** | FLAN (Wei et al., arXiv 2109.01652) | Coins "instruction tuning"; 62-task instruction collection; zero-shot transfer to unseen tasks; beats zero-shot 175B GPT-3 on 20/25 tasks |
| **Mar 2022** | InstructGPT (Ouyang et al., arXiv 2203.02155) | The SFT→RM→RLHF recipe at frontier scale; labelers prefer InstructGPT 85% over GPT-3; spawns ChatGPT (Nov 2022) |
| **Apr 2022** | T0 (Sanh et al., arXiv 2110.08207) | Open-source FLAN-style multitask instruction tuning (BIG-Bench etc.); the "zero-shot became a thing" era for open models |
| **Dec 2022** | Self-Instruct (Wang et al., arXiv 2212.10560) | Bootstrapping instructions from the model itself; GPT-3 self-instructed comes within ~5% of InstructGPT001 on curated expert tasks; unlocks the synthetic-data wave |
| **Mar 2023** | **Alpaca** (Stanford CRFM) | LLaMA-7B tuned on 52k text-davinci-003-generated demonstrations for ~$500; proves open instruction tuning is replicable and cheap |
| **Apr 2023** | **Dolly 2.0** (Databricks) and **OASST1** (OpenAssistant) | 15k fully human-written open dataset + 161k-message multilingual human corpus; the open-data era |
| **May 2023** | **LIMA** (Zhou et al., arXiv 2305.11206) | 1,000 curated examples rival models trained on 52k; "Less Is More for Alignment" — quality > quantity |
| **May 2023** | **DPO** (Rafailov et al., arXiv 2305.18290) | Preference optimisation without reward models or RL; assumes an SFT reference policy — cements SFT as stage 1 of modern alignment |
| **Jul 2023** | Llama 2 Chat (Touvron et al.) | Industry-scale demonstration of the full SFT→RLHF pipeline on open weights; SFT data of ~27k annotations + 1.4M RLHF comparisons (verified in the Llama 2 paper) |
| **2024–2025** | Qwen2.5-Instruct, Llama 3.1 Instruct, etc. | Every frontier open release ships an instruction-tuned variant as the default product; instruction tuning is now the unremarkable standard first stage of any model release |

### 2.4 The History Table

| Year | Work | Contribution |
|---|---|---|
| 2021 | **FLAN** — Wei et al., arXiv 2109.01652 | Named and demonstrated instruction tuning: 62 tasks in instruction form → zero-shot transfer to unseen tasks; beats zero-shot 175B GPT-3 on 20/25 tasks |
| 2022 | **InstructGPT** — Ouyang et al., arXiv 2203.02155 | The SFT→RM→RLHF recipe; 85% labeler preference over GPT-3; proved tuned-small beats base-huge; parent of ChatGPT |
| 2022 | **T0** — Sanh et al. | Open multitask instruction-tuned models; made zero-shot capability accessible to the open-source community |
| 2022 | **Self-Instruct** — Wang et al., arXiv 2212.10560 | Self-generated instruction data; bootstrap pipeline (seed tasks → generate → filter → fine-tune); ~5% gap to InstructGPT001 |
| 2023 | **Alpaca** — Taori et al., Stanford CRFM | 52k synthetic demonstrations; LLaMA-7B tuned for ~$500; kicked off the open instruction-tuning boom |
| 2023 | **Dolly 2.0** — Databricks | databricks-dolly-15k: 15k fully human-written pairs, openly licensed; "first open, commercially viable instruction-tuned LLM" |
| 2023 | **OASST1** — OpenAssistant | 161,443 human messages, ~66k conversation trees, 35 languages, ~13.5k volunteers; the largest open human chat corpus of its era |
| 2023 | **LIMA** — Zhou et al., arXiv 2305.11206 | 1,000 curated examples rival 52k-example models; "Less Is More for Alignment"; quality > quantity |
| 2023 | **DPO** — Rafailov et al., arXiv 2305.18290 | Preference learning without RL; uses the SFT model as reference policy; made stage-2 alignment cheap enough for everyone |
| 2023 | **Llama 2 Chat** — Touvron et al. | Open-weights full-scale SFT→RLHF demonstration; ~27k SFT annotations, 1.4M preference comparisons |

---

## 3. Instruction Datasets: The Canonical Six

Six datasets define the instruction-tuning canon. They split into three lineages: **(a) human-written** (Dolly, OASST1, LIMA's curation), **(b) machine-generated from a teacher model** (Self-Instruct, Alpaca), and **(c) task-collection** (FLAN). Understanding where your data comes from matters more than its size (§3.7).

### 3.1 FLAN — The 62-Task Collection (2021)

- **Size:** 62 datasets, ~4.4M examples in the full FLAN collection (verified in the paper's appendix; the 62-dataset figure is from the paper).
- **Source:** existing NLP benchmark datasets (NLI, QA, reading comprehension, translation, summarisation, etc.), each rendered into instruction form with **multiple template phrasings** (template ensembling).
- **Quality notes:** high *task diversity*, but the instructions are template-generated, not natural user language; data is English-centric and benchmark-derived.
- **Legacy:** the template-ensembling design and "hold out whole task clusters for evaluation" practice are still best practice today.

### 3.2 Self-Instruct — Self-Generated Instructions (2022)

- **Size:** ~52k instructions + input/output pairs in the released GPT-3 run (verified in the paper; the pipeline is what matters, not the exact count).
- **Source:** **bootstrap from the model itself**: (1) seed with 175 human-written instruction tasks; (2) prompt the LM to generate new instructions; (3) filter (deduplicate by similarity, drop instructions that repeat input or are invalid); (4) generate input and output for each instruction; (5) fine-tune the original model on the result.
- **Quality notes:** noisy by construction — machine-generated instructions drift toward the seed distribution, and outputs inherit the teacher's failure modes. The paper reports self-instructed GPT-3 lands within **~5% absolute** of InstructGPT001 on expert-written novel tasks (verified).
- **Legacy:** the entire synthetic-data industry (Alpaca, WizardLM, UltraChat, and most 2023+ commercial data pipelines) descends from this pattern.

### 3.3 Alpaca — The 52k Demonstrations (2023)

- **Size:** **52k** instruction-following demonstrations (verified on the Stanford CRFM page and the `tatsu-lab/stanford_alpaca` repo: "52K instruction-following demonstrations"). Around 40% of examples include an `input` field (context for the task).
- **Source:** Stanford CRFM, March 2023. The 52k instructions were generated in Self-Instruct style using **text-davinci-003** as the teacher; the model was **LLaMA-7B**, fine-tuned with **supervised learning only** (no RLHF).
- **Cost:** ~$500 for data generation (verified claim on the project page) — the number that convinced the world instruction tuning was cheap.
- **Quality notes:** single-turn only; homogeneous style (davinci-003's voice); well-known issues include verbatim memorisation of the teacher's phrasing and some nonsensical instructions. Later analysis (e.g., "AlpacaFarm" evaluations) showed the model underperforms davinci-003 but still captured broad instruction-following behaviour.
- **Legacy:** the template for every "tune an open model on teacher-generated data" project; also the cautionary tale about licensing (the original Alpaca was retracted from distribution after LLaMA's license restrictions).

### 3.4 Dolly — The 15k Human-Written Dataset (2023)

- **Size:** **15,000** high-quality human-generated prompt/response pairs (verified: `databricks-dolly-15k`).
- **Source:** Databricks, April 2023. Written from scratch by **Databricks employees** (not scraped, not machine-generated) across 8 categories: brainstorming, classification, closed QA, generation, information extraction, open QA, summarisation. Licensed under **CC BY-SA 3.0** — one of the first genuinely permissive human-written instruction datasets.
- **Quality notes:** human naturalness and variety; but written by a single company's employees (style homogeneity), single-turn, and the responses are "good enough" rather than frontier quality. Dolly 2.0 (pythia-12b tuned on it) exhibited respectable instruction-following despite the small, non-synthetic dataset — evidence for the quality-over-quantity thesis.

### 3.5 LIMA — The 1k Curated Examples (2023)

- **Size:** **1,000** carefully curated prompts and responses (verified: "LIMA consists of 1,000 carefully curated prompts and responses" — Zhou et al., arXiv 2305.11206).
- **Source:** Meta AI, May 2023. Curated by the authors from community forums (Stack Exchange, wikiHow, Reddit) and manually written, with a **principle-based curation** process: correctness (factually right), style (assistant-like), safety (harmless), and diversity (broad task coverage). Responses were heavily edited or written from scratch by the authors.
- **Quality notes:** the paper's thesis is "**Less Is More for Alignment**": LIMA (65B) tied or beat Alpaca 65B (trained on 52k examples) in human preference evals, and was competitive with DaVinci003 on many prompt distributions — though humans still preferred GPT-4/Claude/Bard. Ablations showed quality *and* diversity both matter; sheer quantity did not.
- **Lesson:** this is the single most cited evidence that **curation beats volume** (§3.7).

### 3.6 OpenAssistant OASST1 — The 161k-Message Multilingual Corpus (2023)

- **Size:** **161,443 messages** across **66,497 conversation trees**, in **35 languages**, with **461,292 quality ratings** (verified on the OpenAssistant/oasst1 dataset card). Note: the "161k" commonly cited is *messages*, not examples — after flattening into instruction–response pairs the usable volume is smaller, and multi-turn trees are the natural unit.
- **Source:** OpenAssistant (LAION-led), April 2023. Collected from ~**13,500 volunteers** worldwide: humans wrote the initial prompts and responses, then *other* humans rated them — a fully human pipeline with quality annotations built in.
- **Quality notes:** unmatched diversity of language, topic, and register; multi-turn trees (good for chat tuning); but volunteer quality is uneven — the quality ratings exist precisely to filter it. Models tuned on OASST1 (e.g., OpenAssistant's own, and many 2023 open models) show broad but sometimes shallow instruction-following.
- **Legacy:** the reference "crowdsourced human chat corpus"; demonstrated both the value and the noise of volunteer data.

### 3.7 The Dataset Comparison Table

| Dataset | Size | Source | Quality notes |
|---|---|---|---|
| **FLAN** (2021) | 62 tasks, ~4.4M examples | Existing NLP benchmarks, template-rendered | Max task diversity; template instructions, not natural user language; English-centric |
| **Self-Instruct** (2022) | ~52k instructions (+ I/O) | Model-generated (bootstrap + filter) | Noisy by construction; drifts to seed distribution; ~5% gap to InstructGPT001 when applied to GPT-3 |
| **Alpaca** (2023) | **52k** | text-davinci-003-generated (Self-Instruct style) | Homogeneous teacher voice; some nonsensical instructions; ~$500 to generate; single-turn |
| **Dolly** (2023) | **15k** | Human-written (Databricks employees), CC BY-SA 3.0 | Natural and varied; single-company style; "good enough" not frontier responses |
| **LIMA** (2023) | **1,000** | Human-curated/edited (principle-based) | Highest per-example quality; correctness + style + safety + diversity; the quality>quantity proof |
| **OASST1** (2023) | **161,443 messages**, ~66k trees, 35 languages | ~13.5k volunteer humans + ratings | Broad multilingual/multi-turn coverage; uneven quality — ratings exist to filter it |

### 3.8 The Dataset Lessons

**Lesson 1 — Quality > quantity, and diversity is the second axis.** LIMA is the cleanest demonstration: 1,000 curated examples matched 52k machine-generated ones in human preference. But the paper's ablations are equally important: *both* quality and diversity drive gains — a large set of redundant high-quality examples underperforms a small set of diverse ones. In practice this means: curate for correctness, edit responses to assistant style, and cover many task types rather than many paraphrases of one task. (The "1k–50k sweet spot" in §4.3 is a heuristic shaped by exactly this evidence — flagged, not a law.)

**Lesson 2 — Decontamination is non-negotiable.** If your SFT data contains (or is templated from) the same benchmarks you later evaluate on — or the same documents your RAG eval set cites — your evaluation is measuring memorisation, not capability. The FLAN paper's design (hold out entire task clusters) is the canonical answer. Practical rules: (a) keep eval benchmarks out of SFT data by construction; (b) run fuzzy/embedding deduplication between SFT data and eval sets; (c) for enterprise data, also scan for PII and for eval-derived text (e.g., leaked benchmark questions). Details in §8.2, and the eval-governance framing in `llm_evaluation_vs_validation_guide.md`.

**Lesson 3 — Know your lineage.** Human-written data is clean but expensive and style-limited; teacher-generated data is cheap and scalable but inherits teacher failure modes and can memorise the teacher's voice; benchmark-derived data is diverse but templated. Production datasets are usually **mixed**: a curated human core + filtered synthetic expansion + domain data (§9).

---
## 4. SFT Mechanics: Templates, Masking, and Training Setup

Instruction tuning is supervised training — but the details of *how* the examples are formatted, *which* tokens contribute to the loss, and *how long* you train determine whether you get an obedient assistant or a parrot.

### 4.1 Chat Templates: The Formats That Make the Model Understand Roles

An instruction dataset is stored as plain JSON (`instruction`, `input`?, `output`); a chat template renders it into the token sequence the model actually trains on. The template is not cosmetic — it teaches the model the **role structure** (who is the system, who is the user, who is the assistant) and must be the **same at training and inference time** (the fine-tuning umbrella makes the same point in §8.4: *use the model's native template*).

**ChatML** (Chat Markup Language) — the most widely used format, published by OpenAI and still the native template of the **Qwen** family (verified: Qwen model cards specify ChatML):

```
<|im_start|>system
You are a helpful banking assistant.<|im_end|>
<|im_start|>user
What is the current balance of account SG1234567?<|im_end|>
<|im_start|>assistant
Your balance is SGD 42,318.75 as of today.<|im_end|>
<|im_start|>user
And the pending transactions?<|im_end|>
<|im_start|>assistant
<|im_end|>
```

Each message is delimited by `<|im_start|>` + role + newline and closed with `<|im_end|>`; roles are `system`, `user`, `assistant` (and `tool` in tool-use variants). The final `assistant` opening with no content marks the generation point at inference.

**Llama-family templates** (verified in the Llama model cards):

- **Llama 2** used the `[INST] … [/INST]` bracket format with optional `<<SYS>>…<</SYS>>` system block.
- **Llama 3** uses header tokens: `<|begin_of_text|><|start_header_id|>system<|end_header_id|> … <|eot_id|>` per turn, with `<|start_header_id|>assistant<|end_header_id|>` marking the response slot.

**The template-family table** (verified against model cards and tokenizer configs):

| Family | System marker | User marker | Assistant marker | Turn end |
|---|---|---|---|---|
| **ChatML** (GPT-4o-era OpenAI, Qwen, OpenChat) | `<|im_start|>system\n…<|im_end|>` | `<|im_start|>user\n…<|im_end|>` | `<|im_start|>assistant\n` | `<|im_end|>` |
| **Llama 2** | `<<SYS>>…<</SYS>>` (inside `[INST]`) | `[INST] … [/INST]` | (after `[/INST]`) | `</s>` |
| **Llama 3 / 3.1** | `<|start_header_id|>system<|end_header_id|>` | `<|start_header_id|>user<|end_header_id|>` | `<|start_header_id|>assistant<|end_header_id|>` | `<|eot_id|>` |
| **ShareGPT** (dataset interchange) | `from: system` fields | `from: human` | `from: gpt` | JSON array rows — a *storage* format, not a token format |

The subtle but critical point: **ChatML and ShareGPT are often confused.** ShareGPT is a JSON interchange format used in open datasets (conversations as `{"from": "human"/"gpt", "value": …}` rows); ChatML is a token-level rendering. Datasets ship in ShareGPT-like JSON; training happens in ChatML/Llama-style tokens. Conversion is mechanical but must be done correctly — a wrong conversion silently corrupts the role structure (§4.5's "verify on a sample batch" rule).

**Practical rules:**

- **Never hand-roll the template** — use `tokenizer.apply_chat_template(...)` (Hugging Face) or the model card's canonical renderer. Hand-built templates are the #1 silent cause of "fine-tuned model talks nonsense."
- **Train and serve with the same template** — a mismatch (e.g., training with ChatML, serving with Llama-3 headers) destroys instruction-following.
- **Include system prompts and multi-turn context in training** — models trained only on single (user→assistant) turns follow system instructions and multi-turn context poorly.
- **Template-consistent special tokens must be added to the tokenizer** if you train a base model (the base tokenizer may not have `<|im_start|>`).

### 4.2 Loss Masking: Response-Only Loss

The single most important mechanic in SFT. If you compute cross-entropy over the *whole* sequence, the model is rewarded for predicting the user's instruction — which is the wrong objective and actively teaches the model to "answer questions by restating them."

**Response-only loss (loss masking):** compute the loss **only over the assistant-response tokens**; mask prompt/system/user tokens (in Hugging Face convention, their labels are set to `-100`, which the loss function ignores). Verified as the default behaviour of the TRL `SFTTrainer` and standard practice in every serious SFT implementation (Axolotl, LlamaFactory, Unsloth).

```
input_ids:    <|im_start|>user\nWhat is a swap?<|im_end|>\n<|im_start|>assistant\nA swap is an OTC derivative...<|im_end|>
labels:       -100  -100  ...  -100  -100  -100  -100  -100  A  swap  is  an  OTC  ...  <|im_end|>
              └──────── masked (no loss) ────────┘   └────────── loss computed here ──────────┘
```

Why it matters: response-only loss is what makes SFT *behavioural* — the gradient signal says "make the assistant's answer more probable," never "make the user's question more probable." Two secondary benefits: (1) training is faster per example (fewer contributing tokens); (2) the model doesn't learn to degenerate into echoing prompts (a real failure mode with unmasked loss on small datasets).

**Related pitfall — answer leakage:** if a training example contains the *next* turn's assistant answer inside the current window (e.g., from careless packing or a dataset bug), the model can learn to copy answers from context. Response-only masking with correct per-turn labels prevents this; always verify label alignment on a sample batch before a long training run.

### 4.3 The Training Setup: Epochs, LR, Batch

Verified norms across the canonical runs (InstructGPT SFT, Alpaca, Llama 2, and current TRL/Axolotl defaults), with the caveat that they are **heuristics, not laws**:

| Hyperparameter | Typical value | Notes |
|---|---|---|
| **Epochs** | **2–3** (Alpaca: 3; InstructGPT SFT: 16 epochs on 13k — with *val loss for early stopping*; Llama 2: 2) | With 1k–50k examples, 1–3 epochs is the norm. More epochs on small sets = memorisation (see §4.4). **Watch validation loss / eval scores, not the training curve.** |
| **Learning rate** | Full FT: **1e-5–2e-5** (with warmup); LoRA: **1e-4–2e-4**; QLoRA: often 2e-4–5e-5 | Cosine schedule with 3–10% warmup is standard. LoRA can tolerate higher LR because few parameters change. |
| **Batch size** | **16–128 effective** (micro-batch 1–8 × gradient accumulation) | Effective batch = micro-batch × accumulation × GPUs. Modest batches work fine for SFT; the loss is smooth. |
| **Sequence length** | 2048–8192 (match your longest response + context) | Truncation or packing policy matters: truncate the *prompt* end, never silently truncate the answer. |
| **Precision** | BF16 (or FP16); 4-bit NF4 base for QLoRA | Cross-ref `fine_tuning_frameworks_comparison_guide.md` §7 for quantization-aware FT. |
| **Optimizer** | AdamW (β₂=0.95), weight decay 0–0.1, grad clip 1.0 | Standard LLM recipe. |
| **Warmup** | 3–10% of steps | Avoids early instability with LoRA adapters initialised at zero. |

**Data scale (flagged heuristic):** the practical SFT sweet spot is **~1k–50k examples** — LIMA works at 1k, Alpaca at 52k, and most enterprise SFT projects land between 2k and 20k. Datasets much larger than ~100k (e.g., 1M+ synthetic chat corpora) typically add marginal instruction-following value per token and mostly serve style/multitask breadth; they also multiply training cost. This range is an empirical observation, not a verified law — treat it as a starting point and let your eval decide (this guide's §8.3).

### 4.4 Overfitting: The Repetition Failure Mode

SFT overfitting looks different from classic overfitting. Training loss keeps falling, but the model degenerates:

1. **Verbatim memorisation / repetition:** the model reproduces training responses word-for-word (including names, numbers, and errors) — dangerous in banking, where a memorised wrong balance or stale rate is a compliance incident. Llama 2's paper explicitly reports the "repetition" degeneration on SFT and relies on RLHF to clean it up.
2. **Template collapse:** the model starts every answer with the same boilerplate ("Certainly! Here is…"), losing the diversity the dataset encoded.
3. **Style lock:** the model imitates the dataset's authorial voice so strongly it ignores novel phrasing in user input.

**Causes and mitigations:**

| Cause | Mitigation |
|---|---|
| Too many epochs on small data | 1–3 epochs; early-stop on held-out eval; if val loss rises while train loss falls, you are memorising |
| Too-small / too-homogeneous dataset | Curate for diversity (LIMA lesson); add multi-turn and varied phrasings; flag: synthetic data amplifies this |
| Too-high LoRA rank for the data | Rank 8–64; rank > 64 rarely helps and overfits small sets (cross-ref `fine_tuning_frameworks_comparison_guide.md` §4.1) |
| Duplicates / near-duplicates in data | Deduplicate before training (exact + fuzzy/embedding) |
| Data errors (wrong answers, leaked answers) | The model will learn errors; fix data, don't train through it |

**The diagnostic:** evaluate on *held-out* instruction tasks and compare generations against the base model — if the tuned model is worse on generic instruction-following but better on the training distribution, you have overfit (this is also why §7 pairs IFEval with task evals: one catches generic degradation, the other catches task gains).

### 4.5 The Mechanics Table

| Aspect | Description | Best practice |
|---|---|---|
| **Chat template** | Renders JSON examples into role-tagged token sequences (ChatML, Llama-3 headers) | Use `apply_chat_template` / the model's native template; same template at train and inference |
| **Loss masking** | Cross-entropy only over assistant-response tokens; prompts labelled `-100` | Response-only loss always; verify labels on a sample batch before long runs |
| **Epochs** | 1–3 typical for 1k–50k examples | Early-stop on eval; watch for repetition/memorisation |
| **Learning rate** | 1e-5–2e-5 (full), 1e-4–2e-4 (LoRA), cosine + warmup | Warmup 3–10%; LoRA tolerates higher LR |
| **Batch size** | 16–128 effective via grad accumulation | Modest batches suffice; match sequence length to data |
| **Precision** | BF16/FP16; 4-bit NF4 for QLoRA | Use QLoRA on single GPU (cross-ref fine-tuning umbrella §4.2) |
| **Overfitting** | Repetition, template collapse, style lock | Curate + dedupe + limit epochs; eval generic and task capability separately |

---

## 5. SFT vs RLHF vs DPO: The Alignment Stack

Instruction tuning (SFT) is the *first* stage of alignment. It is often confused with RLHF, so the distinction is worth being precise about. The umbrella guide covers the algorithms in depth (`fine_tuning_frameworks_comparison_guide.md` §6: TRL, DPO, PPO, ORPO, GRPO); this section positions SFT among them.

### 5.1 SFT vs RLHF

**SFT** (this guide's subject): supervised maximum-likelihood training on (instruction, gold-response) pairs. The data is a *demonstration* of desired behaviour; the objective is "make the gold response probable."

**RLHF** (reinforcement learning from human feedback — InstructGPT's steps 2–3): trains a **reward model** on human *preferences* (ranking of several candidate responses), then optimises the policy against that reward model with PPO, regularised by a KL penalty to the SFT policy. The data is *comparative* (response A > response B); the objective is "produce responses humans prefer, even when no gold answer exists."

| | SFT | RLHF |
|---|---|---|
| **Data** | Demonstrations (one gold response per prompt) | Preferences (rankings of candidate responses) |
| **Objective** | Maximise likelihood of gold response | Maximise learned reward, KL-constrained |
| **What it optimises** | Imitation of good answers | *Preference* — including subjective quality, safety, helpfulness trade-offs |
| **Cost** | Cheap (hours on one GPU with LoRA) | Expensive: reward model training + many policy rollouts (PPO) |
| **Failure modes** | Memorisation, style lock, limited to demonstrated behaviours | Reward hacking, reward-model overfitting, KL collapse (mitigated by the KL term) |
| **When used** | Always — it is stage 1 of every modern pipeline | When preference data exists and behaviour beyond imitation is needed (helpfulness, safety, harmlessness trade-offs) |

The canonical evidence for why SFT alone is not enough: InstructGPT's paper shows the RLHF stages add significant preference gains *on top of* SFT (labelers strongly prefer the PPO model over the SFT model); Llama 2's paper similarly reports SFT models underperform RLHF models on human evals. But the reverse is also true: RLHF without a solid SFT foundation is unstable and underperforms — SFT is the load-bearing stage.

### 5.2 SFT vs DPO

**DPO (Direct Preference Optimization)** — Rafailov, Sharma, Mitchell et al. (Stanford), **2023**, arXiv **2305.18290** (verified): *"Direct Preference Optimization: Your Language Model is Secretly a Reward Model."*

DPO is a *preference* method like RLHF but without the reward model and without RL. Key verified properties:

- It re-parameterises the RLHF objective: the optimal policy's reward is expressed in terms of the policy ratio, so the preference loss becomes a **simple classification loss** on (chosen, rejected) pairs — a Bradley–Terry-style objective on `log πθ(y_w|x)/πref(y_w|x) − log πθ(y_l|x)/πref(y_l|x)`.
- **It needs a reference policy `πref`** — in practice the SFT model (`πref = πSFT`). This is the formal reason SFT is stage 1: DPO *requires* an instruction-tuned starting point.
- It is **stable, lightweight, and needs no rollouts** — a drop-in replacement for the RLHF stage in most modern pipelines (the paper reports it matches or beats PPO-based RLHF on summarisation and dialogue with far less compute).
- Hyperparameter `β` controls how far the policy may move from the reference (KL regularisation).

**SFT vs DPO in one line:** SFT teaches *what a good answer looks like* (imitation); DPO teaches *which of two answers is better* (preference). SFT maximises likelihood of the chosen response; DPO maximises the *margin* between chosen and rejected responses relative to the SFT policy.

### 5.3 The Alignment Pipeline: SFT → RLHF/DPO

The canonical pipeline, verified across InstructGPT, Llama 2, and current open-model recipes:

```
Pre-trained base model
        │
        ▼
   ┌───────────────┐   (instruction, response) demonstrations
   │   STEP 1: SFT │ ────────────────────────────────────────────► instruction-tuned model
   │ (this guide)  │   → learns to follow instructions (the interface)
   └───────────────┘
        │
        ▼  (two optional, complementary stage-2 paths)
   ┌───────────────┐   preference rankings (A > B)
   │  STEP 2: RLHF │   reward model + PPO, KL-constrained ─────────► aligned chat model
   └───────────────┘
   ┌───────────────┐   preference pairs, no RM, no RL
   │  STEP 2': DPO │   single classification loss vs πSFT ──────────► aligned chat model
   └───────────────┘
```

Modern recipes often chain further stages (iterative DPO, KTO, ORPO, GRPO — cross-ref the umbrella guide §6), and some (ORPO) fuse SFT and preference learning into one objective. But the *conceptual* stack is stable: **SFT establishes instruction-following; preference methods refine it toward human values.** For most enterprise deployments, a well-executed SFT with LoRA is 80–90% of the value; preference methods matter most when subjective quality/safety trade-offs dominate.

### 5.4 The Contrast Table

| Stage | Objective | Data | Typical use |
|---|---|---|---|
| **SFT (instruction tuning)** | Maximise likelihood of gold response — learn *how to respond to instructions* | (instruction, response) demonstrations, 1k–50k | Always; the foundation of every assistant/agent; cheap, reliable, format + behaviour transfer |
| **RLHF (PPO)** | Maximise learned reward (human preference), KL-constrained to SFT policy | Reward model trained on preference rankings; policy rollouts | Frontier quality, safety/helpfulness trade-offs; expensive; needs preference data + RL infra |
| **DPO** | Direct preference loss (chosen vs rejected) against πSFT — no RM, no RL | Preference pairs only | Same goals as RLHF at a fraction of cost; the default stage-2 for most teams; requires SFT first |

---

## 6. PEFT for Instruction Tuning: LoRA and Friends

Instruction tuning is the canonical use case for **parameter-efficient fine-tuning (PEFT)**: the datasets are small, the objective is behavioural, and freezing the pre-trained knowledge while training a small adapter is exactly the right inductive bias. Full details of LoRA/QLoRA/DoRA/AdaLoRA live in `fine_tuning_frameworks_comparison_guide.md` §4; this section gives the instruction-tuning-specific view.

### 6.1 Why PEFT Is the Default for Instruction Tuning

- **Cost:** a 7–8B model instruction-tuned with QLoRA fits on a single 24–48GB GPU; full FT of 70B does not.
- **Stability:** LoRA's low-rank updates are gentle — less catastrophic forgetting of pre-trained knowledge, which matters because SFT data is tiny relative to pre-training.
- **Iteration speed:** PEFT runs complete in minutes-to-hours, enabling the eval-driven loop of §8.3 (train → eval → fix data → retrain).
- **Serving flexibility:** adapters merge into the base weights at inference (zero added latency) and can be swapped per use case (one base model, many adapters).

### 6.2 PEFT Practice: LoRA Configs for Instruction Tuning

Verified guidance from the umbrella guide (§4.1) and current tooling defaults (PEFT, TRL, Axolotl, Unsloth):

| Setting | Typical for instruction tuning | Notes |
|---|---|---|
| **Rank `r`** | **16–64** (7–8B models); 8 for very small datasets | Rank > 64 rarely helps; too-high rank overfits small sets |
| **Alpha** | 2× rank (e.g., r=32 → alpha=64) | Fixed scaling convention in PEFT |
| **Target modules** | All attention linears (q, k, v, o) for Llama-3/Qwen; q_proj+v_proj minimum | Modern practice: target all linear layers, incl. MLP, for better instruction-following |
| **Dropout** | 0.05–0.1 | Small dropout helps generalisation on small sets |
| **Base precision** | BF16, or 4-bit NF4 (QLoRA) for single-GPU | QLoRA reaches 93–98% of full-FT quality (umbrella §4.2) |
| **LR** | 1e-4–2e-4, cosine, 3–10% warmup | Higher than full-FT LR because few params change |

**A representative config sketch** (QLoRA on a 7–8B base; full worked example in §9):

```yaml
# QLoRA config for SFT (TRL/Axolotl-style)
base_model: Qwen/Qwen2.5-7B          # base (or instruct) model
load_in_4bit: true                   # NF4 quantization of the frozen base
lora:
  r: 32
  alpha: 64
  dropout: 0.05
  target_modules: [q_proj, k_proj, v_proj, o_proj, gate_proj, up_proj, down_proj]
training:
  epochs: 2
  learning_rate: 2e-4
  lr_scheduler: cosine
  warmup_ratio: 0.05
  per_device_batch_size: 4
  gradient_accumulation: 8            # effective batch 32 (single GPU)
  bf16: true
  max_seq_len: 4096
  loss_masking: response_only         # mask prompt tokens (labels = -100)
```

### 6.3 The PEFT Table

| Method | Params trained | Memory vs full FT | Quality vs full FT | Best for instruction tuning? |
|---|---|---|---|---|
| **LoRA** | 0.1–1% | 70–90% less | 95–99% | **Default choice** — sweet spot of cost/quality; adapters merge at inference |
| **QLoRA** | 0.1–1% (4-bit base) | 95–98% less | 93–98% | **Single-GPU instruction tuning** — the workhorse for 7–13B models |
| **DoRA** | 0.1–1% | 70–90% less | 96–100% | When you want LoRA's cost with full-FT closeness |
| **AdaLoRA** | 0.1–1% (adaptive rank) | 70–90% less | 96–100% | When you want to skip rank tuning |
| **Full FT** | 100% | baseline | 100% | Frontier-scale releases; overkill and risky for 1k–50k example SFT |

**Rule of thumb:** start with **QLoRA (r=32, alpha=64, all attention+MLP linears)** on your 7–8B base, 2 epochs, 2e-4 LR, and evaluate. Only escalate to full FT (or DoRA) if eval shows the adapter is the bottleneck — which, with a well-curated dataset, is uncommon for instruction-following behaviour.

---
## 7. Instruction-Following Evals: IFEval and Beyond

How do you know an instruction-tuned model actually *follows instructions*? Traditional benchmarks (MMLU, GSM8K) measure knowledge and reasoning, not obedience to constraints. Instruction-following needs its own evaluation — and it is the *first* eval you should run after SFT, because instruction-following is exactly what the stage changed.

### 7.1 IFEval: The Verifiable-Instruction Benchmark

**IFEval (Instruction-Following Evaluation)** — Zhou, Lu, Mishra et al. (Google), **November 2023**, arXiv **2311.07911** (verified): *"Instruction-Following Evaluation for Large Language Models."*

The core idea: IFEval focuses on **"verifiable instructions"** — constraints a program can check automatically — such as *"write in more than 400 words,"* *"mention the keyword 'AI' at least 3 times,"* *"respond in JSON with keys X and Y,"* *"end your response with a specific phrase,"* *"include 2 bullet points,"* *"write in Spanish."* Because the checks are mechanical, evaluation is **cheap, deterministic, and reproducible** — no LLM judge, no human annotator.

Verified design details:

- The released benchmark contains **541 prompts**, each bundling 1–5 verifiable instructions (25 instruction types across 9 categories: punctuation, keywords, language, length, detectability, format, forbidden words, JSON, others).
- **Two levels of check strictness:** *loose* (e.g., word count check tolerating minor formatting) and *strict* (exact).
- **Two reported metrics:** **prompt-level accuracy** (all instructions in a prompt satisfied) and **instruction-level accuracy** (each instruction independently satisfied). Instruction-level is the more granular signal for debugging which constraint types fail.
- Known property (acknowledged by the authors): models can game IFEval by learning template behaviours; it measures *compliance with explicit constraints*, not general quality. Treat it as necessary, not sufficient.

**Why IFEval is the right post-SFT smoke test:** SFT trains the model to satisfy constraints (formats, lengths, keywords) — IFEval measures exactly that, and it is cheap enough to run on every candidate checkpoint during iteration (§8.3). It also acts as a **regression guard**: instruction tuning must not *degrade* constraint-following, and base models typically score far below tuned ones — a quick sanity check that the SFT actually took.

### 7.2 Judge-Based Complements: AlpacaEval and MT-Bench

IFEval measures *obedience to explicit constraints*; it says little about whether responses are *useful, correct, and well-formed* on open instructions. The standard complements are **LLM-as-judge** leaderboards:

- **AlpacaEval** (Tulu/Lucy Smith et al., 2023): ~800 instructions; a strong judge model (GPT-4 in AlpacaEval 2.0) compares the candidate against a reference (text-davinci-003, later GPT-4-Turbo) and reports a **win rate**. Cheap, widely cited, but known to favour verbosity and to be gameable — treat win rates as relative signal, not absolute quality.
- **MT-Bench** (Zheng et al., 2023): 80 multi-turn questions across 8 categories (writing, roleplay, reasoning, math, coding, extraction, STEM, humanities); GPT-4 judges each turn on a 1–10 rubric; produces an average score. Multi-turn structure makes it a better probe of *conversational* instruction-following than single-turn sets.
- **Generalist evaluation (AGIEval, etc.)**: for the reasoning/knowledge side, see the umbrella guide §10.2 and `llm_evaluation_frameworks_guide.md` §4 for metric categories.

**Practice:** run IFEval (mechanical obedience) + AlpacaEval/MT-Bench-style judge evals (subjective quality) as the automated pair, and reserve human SME review for the final gate (§7.4). Judge-based evals inherit the judge's biases — a known limitation, verified across the evaluation literature; never let an LLM judge be the *only* quality signal in a regulated context (cross-ref `llm_evaluation_vs_validation_guide.md`).

### 7.3 Eval Practice: Instruction-Following Accuracy

Building an instruction-following eval for *your* model means more than running IFEval. The practice that works (and the reasoning behind it):

1. **Measure constraint compliance, not just quality.** For each constraint class you care about — output format (JSON, XML, markdown), length limits, language, keyword requirements, "do not mention X," tool-call schemas — build 20–100 prompt templates and check them with **code, not judges**. JSON parse + schema validation, regex for forbidden words, word/char counts. This is "verifiable instructions" applied to your domain.
2. **Pair with a generic benchmark.** IFEval + a generic eval (e.g., MMLU for knowledge regression, GSM8K for reasoning — cross-ref the fine-tuning umbrella §10.2 for the regression-testing argument) tells you whether SFT improved obedience without destroying capability.
3. **Add domain-specific task evals** (§7.4) for the actual job, and **human review** for subjective quality (helpfulness, tone, safety) — none of the automated metrics capture all of it.
4. **Track the delta vs the base model.** The meaningful signal is *change*: base → SFT on the same eval suite. A tuned model that matches the base on generic evals while winning on domain tasks is a success; one that wins both but memorises training responses is overfit (see §4.4).

### 7.4 Task Evals: The Domain Layer

Instruction-following is necessary but not sufficient — the model must also *do the task*. Domain evals measure the job, not the obedience:

- **Domain benchmarks:** for banking (the §9 example): intent classification accuracy, account/transaction QA against a golden set, product-explanation correctness (checked against a controlled glossary), regulatory-question compliance, tool-call extraction accuracy (arguments parsed into the right JSON schema).
- **Golden-set evaluation:** 200–1,000 (prompt, gold response) pairs written by domain experts, scored by exact/fuzzy match, LLM-as-judge, or rubric. This is the highest-signal eval for enterprise SFT. The framework tooling (DeepEval, G-Eval, RAGAS for RAG tasks) is covered in `llm_evaluation_frameworks_guide.md`; the validation-governance framing (when does an eval count as *validation evidence* for model risk) is in `llm_evaluation_vs_validation_guide.md`.
- **Adversarial/robustness evals:** prompt-injection resistance (cross-ref `prompt_injection_guide.md`), jailbreak attempts, out-of-scope requests (should refuse cleanly), ambiguous instructions. SFT data that includes good refusals measurably improves this.
- **Human evaluation on a sample:** 50–100 cases reviewed by domain SMEs, especially where automated metrics are weak (tone, trustworthiness, regulatory nuance). Costly, but it is the ground truth the automated layer approximates.

### 7.5 The Eval Table

| Eval | Method | Use |
|---|---|---|
| **IFEval** | 541 prompts, ~25 verifiable instruction types, programmatic checks; prompt-level & instruction-level accuracy (loose/strict) | Generic constraint-following smoke test; regression guard across checkpoints |
| **Instruction-following accuracy (custom)** | Code-checked constraint templates for your formats (JSON schema, length, forbidden words, tool-call schemas) | Measures the constraints *your* product actually needs |
| **Domain task evals** | Golden sets + task metrics (intent accuracy, QA F1, tool-call schema match); LLM-as-judge for open tasks | Measures whether the model does the job, not just obedience |
| **Generic capability evals** | MMLU, GSM8K, etc. | Regression check — SFT must not destroy knowledge/reasoning |
| **Human SME review** | 50–100 sampled cases, rubric-scored | Ground truth for subjective quality, tone, safety, regulatory nuance |
| **Adversarial evals** | Prompt-injection / jailbreak / out-of-scope suites | Boundary behaviour — refusal quality and instruction/data separation |

---

## 8. Practice: Curation, Decontamination, Iteration

The field's accumulated SFT practice reduces to three moves: **curate the data, decontaminate it, iterate on eval**. Everything else is details.

### 8.1 Dataset Curation: Quality Is a Process

Verified synthesis of the LIMA lesson (§3.5) and industry practice:

- **Write or edit for correctness first.** Every wrong answer in the SFT data becomes a confidently repeated wrong answer. Domain SMEs must review the response side of any domain data.
- **Normalise style to the assistant you want.** Edit responses to the target tone (concise, structured, honest-about-uncertainty). The model will imitate the dataset's voice — that is the point, so choose the voice deliberately.
- **Balance coverage, not volume.** Cover task types (QA, summarisation, extraction, tool calls, refusal) and domains proportionally to usage; avoid 90% of data being one task.
- **Include the hard cases.** Ambiguous instructions, multi-step tasks, refusals, out-of-scope requests, format-demanding requests. These teach behaviour generic examples don't.
- **Deduplicate aggressively.** Exact + fuzzy (embedding similarity) dedup; near-duplicates inflate apparent size and accelerate memorisation.
- **Multi-turn and system-prompt variety.** If the product uses system prompts, vary them in training data.
- **Fix data, don't train through it.** Error analysis (which cases fail) should point back to data gaps; each iteration of §8.3 should grow the dataset's *coverage of failures*, not its volume.

### 8.2 Decontamination: Keep the Eval Honest

- **Definition:** ensuring training data does not overlap with evaluation data — so evals measure generalisation, not memorisation.
- **Why it matters in practice:** a model that saw GSM8K solutions (or your golden eval set, or benchmark questions scraped into "instruction data") scores brilliantly on evals and collapses in production. The FLAN paper's held-out-cluster design (§2.1) is the canonical method: **remove the eval material from the SFT data by construction, then verify by search.**
- **Implementation checklist:**
  1. Build the eval sets first; **exclude their sources** from SFT collection (benchmark files, eval-generation prompts, docs cited by eval questions).
  2. Run fuzzy and embedding-based similarity between SFT data and eval sets; drop flagged near-duplicates (or quarantine them into a "contamination probe" set you use to *measure* leakage).
  3. For enterprise data, also scan for **PII** (banking: account numbers, customer names, card data — see §9) and for copyrighted/confidential content the model must not memorise.
  4. Re-run the checks whenever the dataset grows; contamination is a drift problem, not a one-time fix.
- **Cross-references:** the eval-governance framing in `llm_evaluation_vs_validation_guide.md`; RAG-eval methodology (which has the same contamination concern for corpus-vs-question overlap) in `rag/rag_evaluation_methodology_guide.md`.

### 8.3 Iteration: The Eval-Driven Loop

SFT is not a single run; it is a loop. The loop that works:

```
        ┌──────────────────────────────────────────────────┐
        ▼                                                  │
baseline eval → SFT (LoRA, small) → eval (IFEval + task) → error analysis
   (base model)      (hours)          (same suite!)       (which cases fail, why)
                                                              │
                                                    fix data / add hard cases
                                                    or adjust config, retrain ─┘
```

Rules that make the loop effective:

1. **Freeze the eval suite before training begins** (base-model baseline included). If you change the eval while iterating, you can't compare runs.
2. **Prefer data changes over config changes.** If eval failures cluster on a task type or format, the fix is more/better data for that cluster — not a higher LR or more epochs. Config changes (rank, epochs, LR) are for global symptoms (overfit, instability).
3. **Keep runs small and fast** (QLoRA, 1–2 epochs, a few thousand examples) — iteration speed is the real competitive advantage; the final run can be scaled up once the data is right.
4. **Log everything** (data version, config, eval scores) — see `fine_tuning_frameworks_comparison_guide.md` §9.5 for experiment tracking.
5. **Regression-test the base.** Every iteration should confirm generic capability didn't drop (§7.3).

### 8.4 The Practice Table

| Practice | Rationale | Implementation |
|---|---|---|
| **Curate for correctness + style** | The model imitates the dataset; errors and tone become the product | SME review of responses; edit to target voice; balanced task coverage |
| **Cover hard cases** | Behaviour transfers from examples; easy data teaches little | Include refusals, multi-step, ambiguous, out-of-scope, format-heavy prompts |
| **Deduplicate** | Near-duplicates inflate size and accelerate memorisation | Exact + embedding-similarity dedup before training |
| **Decontaminate** | Evals must measure generalisation, not memorisation | Eval sets built first; fuzzy/embedding overlap checks; PII scans |
| **Eval-driven iteration** | Data, not config, is the main lever; speed wins | Frozen eval suite; small fast runs; error analysis → data fixes → retrain |
| **Regression-check the base** | SFT must add behaviour, not destroy capability | IFEval + MMLU/GSM8K-style suite before/after each run |

---

## 9. Worked Example: Instruction-Tuning a Banking Assistant

### 9.1 The Scenario

A bank (Cymbal Bank-style wholesale bank — the familiar internal-assistant scenario from `../../banking/core_banking_processes_guide.md` and `../../banking/full_stack_banking_guide.md`) wants an **internal assistant** for relationship managers and ops staff: answer product questions (trade finance, FX, cash management), extract structured data from messages (SWIFT MT/ISO 20022), summarise documents, and refuse anything out of scope — all in a controlled, auditable way. The team's constraint: **no external APIs** (data never leaves the bank), one GPU, and a **7–8B open model**. That constraint makes instruction tuning with QLoRA the natural choice (cross-ref `on_prem_llm_deployment_guide.md` for deployment).

**Decision point:** start from a *base* model (Qwen2.5-7B base) and do full instruction tuning, or from the *instruct* variant and do continued instruction tuning? For an enterprise domain assistant, **continued SFT on the instruct variant is the pragmatic default** (faster convergence, less risk); starting from the base demonstrates the technique end-to-end and maximises behavioural control. The worked example starts from the **base model** to show the full SFT stage; the same recipe works on the instruct variant with half the data. (Note: Qwen2.5-Instruct and Llama 3.1 Instruct are themselves SFT+preference-tuned variants — verified in their model cards; "base vs instruct" is a continuum, not a binary.)

### 9.2 The Dataset

**Size target: ~8,000 curated examples** (within the flagged 1k–50k heuristic). Composition:

| Category | Count | Example instruction → response |
|---|---|---|
| Product QA (trade finance, FX, cash) | 2,500 | "Explain a letter of credit in three sentences." → concise definition + when it's used |
| Structured extraction (tool-call style) | 1,500 | "Extract counterparty, amount, currency, value date from this MT103: …" → `{"counterparty": "...", "amount": ..., ...}` (JSON, schema enforced) |
| Document summarisation | 1,000 | "Summarise this ISDA amendment in 5 bullets, flagging payment changes." → 5 bullets + flags |
| Process/how-to (internal) | 1,000 | "What are the steps to open an import LC in the portal?" → numbered steps |
| Refusals & out-of-scope | 800 | "Give me a personal mortgage quote." → policy-compliant refusal redirecting to retail channels |
| Ambiguous / multi-step / adversarial | 800 | "Are there any fees? (no context)" → asks for clarification rather than guessing |
| General instruction-following (safety net) | 400 | A small IFEval-style mix so generic obedience is maintained |

**Curated, not scraped:** responses written/edited by product SMEs and compliance reviewers (per §8.1); system prompts varied; multi-turn versions of the top 1,000 cases included. **Decontaminated:** the 400-case eval golden set (§9.4) was built *first*, and its source documents were excluded from the SFT corpus; embedding-similarity checks run on every dataset revision; PII scrubbed (no real account numbers/names — synthetic placeholders).

### 9.3 The SFT: LoRA Fine-Tune (Config Sketch)

Using the §6.2 recipe, on a single 48GB GPU (or 24GB with 4-bit):

```yaml
base_model: Qwen/Qwen2.5-7B            # base model; ChatML native template
dataset: banking_sft_v3.jsonl          # 8,000 examples, ChatML-rendered
lora:
  r: 32; alpha: 64; dropout: 0.05
  target_modules: [q_proj, k_proj, v_proj, o_proj, gate_proj, up_proj, down_proj]
training:
  epochs: 2; learning_rate: 2e-4; scheduler: cosine; warmup_ratio: 0.05
  per_device_batch_size: 4; gradient_accumulation: 8   # effective 32
  bf16: true; max_seq_len: 4096; packing: false
  masking: response_only               # labels = -100 on prompts (verified default in TRL SFTTrainer)
```

Expected runtime: ~2–4 hours on one A100/H100. The run is repeated with the eval loop (§8.3): three iterations were needed — iteration 1 exposed weak JSON-schema adherence on extraction (data fix: more schema-varied examples); iteration 2 exposed refusal overreach (the model refused in-scope product questions — data fix: rebalanced refusal/QA ratio and softened refusal phrasings); iteration 3 passed the eval gates.

### 9.4 The Eval: IFEval + Task Evals (Illustrative Results)

**Eval suite** (frozen before training): IFEval (generic obedience) + 400-case banking golden set (task) + JSON-schema compliance on extraction + 100-case adversarial set (injection, out-of-scope) + MMLU subset (regression guard). The table below is **illustrative of the shape of results** — representative of published SFT deltas, but **not measured in this document** (flagged).

| Eval | Base Qwen2.5-7B | After SFT | Signal |
|---|---|---|---|
| IFEval instruction-level (loose) | ~55% | **~78%** | Obedience to explicit constraints up sharply |
| Banking golden set (task accuracy) | ~48% | **~86%** | Domain task competence (the main win) |
| JSON-schema compliance (extraction) | ~35% | **~92%** | Format adherence — the tool-call enabler |
| Refusal correctness (adversarial set) | ~60% | **~88%** | Boundary behaviour |
| MMLU subset (regression guard) | ~62% | **~61%** | No capability regression (within noise) |

**How the results were read:** IFEval + schema compliance proved the *instruction-following* stage took; the golden set proved the *domain* stage took; the MMLU flat line ruled out catastrophic forgetting; the adversarial set flagged refusal tuning as the next iteration's focus. The model was then deployed behind the bank's guardrails (cross-ref `llm_guard_models_guide.md`), with the eval suite wired into CI as the regression gate (cross-ref `llm_evaluation_frameworks_guide.md` §15 for CI/CD integration patterns), and human SME review retained for release sign-off per `llm_evaluation_vs_validation_guide.md`.

### 9.5 The Lessons

1. **The data quality decides.** The single biggest score jump (golden set 48% → 86%) came from the curated, SME-reviewed dataset — not from the LoRA config. The two failed iterations were *data* problems (schema variety, refusal balance), fixed by data changes, exactly as §8.3 predicts.
2. **Start from the instruct variant for speed; from the base for control.** Both work; the choice is risk vs convergence time. Either way the mechanics are identical.
3. **Decontamination was load-bearing for trust.** Because the golden set was built first and overlap-checked, the 86% figure is credible to the model-risk reviewers — in a regulated bank, an eval that could be memorisation is worthless.
4. **Evals must be frozen, paired, and continuous.** IFEval alone would have missed the domain win; the golden set alone would have missed the obedience regression risk. Both, in CI, are the deployment gate.
5. **Instruction tuning is the base layer, not the whole system.** The assistant still needs RAG for live product data (cross-ref `rag/rag_frameworks_comparison_guide.md`), guardrails, and monitoring — SFT gives it the *behaviour*; the platform gives it the *context and control* (cross-ref `autonomous_agents_guide.md` for the agentic extension).

---

## 10. Summary: Instruction Tuning in One Page

**Instruction tuning is the supervised fine-tuning (SFT) stage that teaches a base model to follow instructions** — training on (instruction, response) pairs so the model responds to requests instead of continuing text. It is the *first and load-bearing stage of alignment*: pre-training supplies knowledge and language; instruction tuning supplies the behavioural interface; RLHF/DPO (which require an SFT model as their reference) refine behaviour toward human preferences.

**The essentials, in five lines:**

- **Data:** 1k–50k (instruction, response) pairs — FLAN (62 templated tasks), Self-Instruct (self-generated), Alpaca (52k synthetic), Dolly (15k human), LIMA (1k curated — *quality > quantity*), OASST1 (161k human messages, 35 languages). Curate, deduplicate, and decontaminate.
- **Mechanics:** render examples with the model's chat template (ChatML / Llama headers), compute **response-only loss** (mask the prompt), train 1–3 epochs at 1e-5–2e-5 (full) or 1e-4–2e-4 (LoRA), watch for repetition/memorisation overfitting.
- **Method:** QLoRA on a 7–8B base is the default — 93–98% of full-FT quality on one GPU, hours not days, adapters merge at inference.
- **Evaluation:** IFEval (verifiable instructions) + custom constraint checks + domain golden sets + regression guards, run as a frozen suite before/after every run.
- **Practice:** the eval-driven loop — small fast runs, error analysis, fix the *data*, retrain. The worked banking example (§9) is the template: 8k curated banking pairs, QLoRA on Qwen2.5-7B, IFEval + golden-set gates, and a model whose behaviour — not just its knowledge — is now the bank's own.

**The final word:** instruction tuning is the stage that turns a text predictor into a *worker* — the stage that teaches the model to follow. Everything above it (preference tuning, agents, RAG, guardrails) assumes it; everything below it (pre-training) cannot do its job alone. Get the data right, and the stage is cheap, fast, and decisive: with a few thousand curated examples and a single GPU, a 7B model becomes an obedient, domain-competent assistant. Get the data wrong, and no amount of compute, rank, or epochs will save it.

---

## 11. Glossary

| Term | Definition |
|---|---|
| **Instruction tuning** | Fine-tuning a language model on (instruction, response) pairs so it learns to respond to natural-language requests; the technique named by FLAN (Wei et al., 2021) |
| **SFT (supervised fine-tuning)** | Training a model with standard supervised maximum-likelihood loss on labelled examples; instruction tuning is SFT on instruction–response pairs |
| **Supervised fine-tuning** | The general mechanism: continue training a pre-trained model on labelled data, maximising the likelihood of gold outputs |
| **Base model** | A pre-trained model trained only for next-token prediction on raw text; no instruction-following behaviour |
| **Instruction-following** | The behaviour of satisfying a user's stated request and constraints (format, length, content); the capability instruction tuning instils |
| **InstructGPT** | OpenAI's 2022 model (Ouyang et al., arXiv 2203.02155) trained with SFT→RM→RLHF; labelers preferred it over GPT-3 85% of the time; parent of ChatGPT |
| **Ouyang** | Long Ouyang, first author of the InstructGPT paper ("Training language models to follow instructions with human feedback," 2022) |
| **FLAN** | "Finetuned Language Models Are Zero-Shot Learners" (Wei et al., 2021, arXiv 2109.01652); coined instruction tuning; 62 templated tasks |
| **Wei** | Jason Wei, first author of the FLAN paper (and of chain-of-thought prompting) |
| **Self-Instruct** | Wang et al., 2022 (arXiv 2212.10560): bootstrapping instruction data from the model itself — generate, filter, fine-tune |
| **Alpaca** | Stanford CRFM, 2023: LLaMA-7B tuned on 52k text-davinci-003-generated demonstrations (~$500); the open instruction-tuning boom starter |
| **LIMA** | "Less Is More for Alignment" (Zhou et al., 2023, arXiv 2305.11206): 1,000 curated examples rival 52k; the quality>quantity evidence |
| **Dolly** | Databricks 2023: databricks-dolly-15k, 15k fully human-written pairs (CC BY-SA 3.0); Dolly 2.0 was the "first open commercially viable instruction-tuned LLM" |
| **OpenAssistant** | LAION-led 2023: OASST1, 161,443 human messages in ~66k trees, 35 languages, ~13.5k volunteers, 461k quality ratings |
| **Chat template** | The rendering that turns JSON into role-tagged token sequences (system/user/assistant); must match between training and inference |
| **ChatML** | Chat Markup Language: `<|im_start|>role\n…<|im_end|>` delimiters; OpenAI's published format, native to Qwen-family models |
| **Loss masking** | Computing the training loss only over response tokens, masking prompts (labels = −100 in HF convention) |
| **Response-only loss** | The standard SFT loss-masking scheme: only assistant-response tokens contribute gradient |
| **Epoch** | One full pass over the training set; SFT typically uses 1–3 |
| **Learning rate** | Step size of weight updates; ~1e-5–2e-5 (full FT), ~1e-4–2e-4 (LoRA) |
| **Overfitting** | In SFT: verbatim memorisation, repetition, template collapse; watch eval, not training loss |
| **RLHF** | Reinforcement learning from human feedback: reward model on preference rankings + PPO, KL-constrained; InstructGPT's stages 2–3 |
| **DPO** | Direct Preference Optimization (Rafailov et al., 2023, arXiv 2305.18290): preference loss without RM or RL; requires an SFT reference policy |
| **Rafailov** | Rafael Rafailov, first author of the DPO paper ("Your Language Model is Secretly a Reward Model," 2023) |
| **PEFT** | Parameter-efficient fine-tuning: train a small subset (adapters) while freezing the base; LoRA/QLoRA/DoRA/AdaLoRA |
| **LoRA** | Low-Rank Adaptation (Hu et al., 2021): low-rank update matrices on attention/MLP weights; ~0.1–1% of parameters; merges at inference |
| **IFEval** | Instruction-Following Evaluation (Zhou et al., Google, 2023, arXiv 2311.07911): 541 prompts with verifiable instructions, programmatic checks |
| **Decontamination** | Ensuring training data doesn't overlap eval data, so evals measure generalisation not memorisation |
| **Dataset curation** | The process of selecting, writing, editing, deduplicating, and balancing instruction data — the dominant lever on SFT quality |

---

*End of guide. Related coverage: `fine_tuning_frameworks_comparison_guide.md` (umbrella), `llm_evaluation_frameworks_guide.md` (eval tooling), `llm_evaluation_vs_validation_guide.md` (validation governance), `autonomous_agents_guide.md` (agents on top of instruction-following), `prompt_injection_guide.md` (instruction-boundary attacks), `rag/rag_evaluation_methodology_guide.md` (RAG evals), `../../banking/core_banking_processes_guide.md` (banking domain). Verification notes: historical facts (papers, arXiv IDs, dates, dataset sizes) verified against primary sources at the time of writing; the 1k–50k data-scale range, the §9.4 eval table, and runtime estimates are flagged heuristics/illustrations, not measured results.*
