# Fine-Tuning a Mixture of Experts — Every Fine-Tuning Run Is Also a Routing Run

> **Byline:** Jack Liu Shurui, Solution Architect  
> **Repo:** personal research library · **Series:** LLM/AI Engineering & Operations · **Domain:** AI Engineering · Fine-Tuning  \
> **Date:** October 2026 · **Sources read:** 2026-10-14

> **Standfirst.** This repository owns fine-tuning four times over and the mixture-of-experts (MoE) mechanism almost not at all. Between them the four existing guides explain what fine-tuning is, how PEFT works, how instruction tuning is run, how a model is compressed, and how a model is deployed on premises — and not one of them asks the question that decides whether a fine-tune of a *sparse* model will hold: **which experts received gradient from the data, and did the router ever send the domain's tokens to them?** A fine-tuning run on a sparse model is not one optimisation; it is two. The adapted parameters are tuned by the loss, and the *routing* is tuned — or frozen, or left quietly drifting — and only the intersection of "the experts this data reached" and "the experts this task needs" is what the fine-tune actually produced. What is unowned, and therefore what this guide holds, is the sparse-specific layer of fine-tuning: what changes when the feed-forward block is not one network but a committee, when the tunable set is data-dependent, and when the evaluation set can look healthy while the routing underneath it has collapsed. Everything the sibling guides own — PEFT methods, data preparation, framework choice, quantisation, the serving stack — is cited here by name and used, never restated.

> **How to read it.** Section 1 is the decoder, the thesis, and the boundary declared by name against the four owner guides. Section 2 is the MoE mechanism itself — the one thing this repository was missing — built entirely from papers and published configurations with their dates. Section 3 is the load-balancing problem, because it is the thing that later breaks under a fine-tune. **Section 4 is the load-bearing section**: why fine-tuning a sparse model is not fine-tuning a dense one, in mechanism, in consequence, and in the honest counter-case where the distinction is theoretical. Section 5 is the method choice, cross-referenced rather than re-derived, considered only for *which modules* get adapted. Section 6 is targeting the experts; Section 7 is the data question in the MoE's own terms — does the training data actually route to the experts being adapted. Section 8 is infrastructure; Section 9 is how to evaluate a sparse model and its routing together. Section 10 is the failure modes as symptom / cause / guardrail. Section 11 is the open models, factually, with no quality ranking. Section 12 is the regulated and on-premises angle, cross-referenced. Section 13 is the Cymbal Bank worked example. Sections 14–15 are the anti-patterns and the claims audit; Section 16 is the honest negatives, the glossary, the cross-references and the closing summary. Architecture numbers change between model versions, so every one is cited to a primary source with the date it was read, and where a figure could not be retrieved the guide prints no figure.

> **Verification policy.** Facts marked **(verified)** were checked against a primary source during writing on 2026-10-14 — an arXiv paper, a published model card, a model's published `config.json`, or a framework's own documentation. Where a figure is version-dependent, the source and its date are given so that a number taken from one model checkpoint is never read as a property of "MoE". Everything else carries a label: **(vendor claim)** for a statement a vendor makes about its own product; **(flagged)** where a claim is repeated but not traceable to a measurement; **(unverified-flag)** where a page could not be extracted; **(negative finding)** where a search pass established that something does *not* appear in the sources checked. No citation in this guide is written from memory, and no parameter count, benchmark score or throughput number is asserted without a primary source and a date.

---

**Table of contents**

1. [Overview: The Decoder, the Thesis, and the Boundary Declared](#1-overview-the-decoder-the-thesis-and-the-boundary-declared)
2. [What a Mixture of Experts Is — The Mechanism This Repository Was Missing](#2-what-a-mixture-of-experts-is--the-mechanism-this-repository-was-missing)
3. [The Load-Balancing Problem](#3-the-load-balancing-problem)
4. [Why Fine-Tuning a Sparse Model Is Not Fine-Tuning a Dense One](#4-why-fine-tuning-a-sparse-model-is-not-fine-tuning-a-dense-one)
5. [The Method Choice, Cross-Referenced Not Re-Derived](#5-the-method-choice-cross-referenced-not-re-derived)
6. [Targeting the Experts](#6-targeting-the-experts)
7. [The Data Question, in the MoE's Terms](#7-the-data-question-in-the-moes-terms)
8. [The Infrastructure](#8-the-infrastructure)
9. [Evaluating a Sparse Model and Its Routing](#9-evaluating-a-sparse-model-and-its-routing)
10. [The Failure Modes](#10-the-failure-modes)
11. [The Open Models, Factually](#11-the-open-models-factually)
12. [The Regulated and On-Premises Angle](#12-the-regulated-and-on-premises-angle)
13. [The Cymbal Bank Worked Example](#13-the-cymbal-bank-worked-example)
14. [The Anti-Patterns](#14-the-anti-patterns)
15. [The Claims Audit](#15-the-claims-audit)
16. [What Could Not Be Verified, the Glossary, the Cross-References and the Closing Summary](#16-what-could-not-be-verified-the-glossary-the-cross-references-and-the-closing-summary)

---
## 1. Overview: The Decoder, the Thesis, and the Boundary Declared

### 1.1 The decoder terms

A sparse model, in this guide, means a **mixture of experts (MoE)**: a transformer in which the single feed-forward network (FFN) of some or all layers is replaced by a *set* of feed-forward networks, only a few of which run for any given token. The pieces are:

- **The expert.** One feed-forward sub-network — in the transformer lineage, an FFN with its own weights — that competes with its siblings to process a token. In the founding formulation the layer held "up to thousands of feed-forward sub-networks" (Shazeer et al., *Outrageously Large Neural Networks*, arXiv 1701.06538, submitted 23 Jan 2017). **(verified)**
- **The router / gate.** A small learned function that scores each token against each expert and picks which experts run. It is "a trainable gating network" whose output "determines a sparse combination of these experts to use for each example" (1701.06538). **(verified)**
- **Top-k.** How many experts a token is sent to. Mixtral routes each token to two of eight experts per layer; Switch Transformers simplified this to top-1 (one expert per token). **(verified)**
- **The shared expert.** An expert (or experts) that runs for *every* token, carrying common knowledge, isolated from the routed experts. DeepSeekMoE introduced "isolating $K_s$ experts as shared ones, aiming at capturing common knowledge and mitigating redundancy in routed experts" (arXiv 2401.06066, 11 Jan 2024). **(verified)**
- **The fine-grained expert.** Splitting the conventional experts into more, smaller ones so a token can pick a more flexible combination. DeepSeekMoE's first strategy is "finely segmenting the experts into $mN$ ones and activating $mK$ from them" (arXiv 2401.06066). **(verified)**
- **The load-balancing loss.** An auxiliary term added to the training loss to push the router toward using experts evenly, so no expert starves. GShard defines "an auxiliary loss term $\ell_{aux}$ to enforce this constraint … added to the overall loss function … with a constant multiplier $k$" (arXiv 2006.16668, 30 Jun 2020; full text). **(verified)**
- **The capacity factor.** A per-expert ceiling on how many tokens an expert may process. Switch Transformers defines it precisely: "each expert has a fixed batch size of (total_tokens / num_experts) × capacity_factor. If the tokens are unevenly dispatched then certain experts will overflow … [and] tokens … are dropped" (arXiv 2101.03961, 11 Jan 2021; full text). **(verified)**
- **Sparse activation.** Only the selected experts compute, so the model's *total* parameters exceed the parameters used per token.
- **Total vs active parameters.** The two numbers that describe a sparse model's size and its cost. DeepSeek-V3 is "671B total parameters with 37B activated for each token" (arXiv 2412.19437, 27 Dec 2024). **(verified)**
- **Expert parallelism (EP).** Distributing the experts across GPUs, so each device holds a subset. Megatron's documentation: EP "distributes experts of a Mixture of Experts (MoE) model across GPUs … applied to only the expert layers" (Megatron Bridge *Parallelisms Guide*, retrieved 2026-10-14). **(verified)**
- **Upcycling.** Reusing a trained dense checkpoint to initialise a sparse model rather than training the sparse model from scratch. "Sparse upcycling — a simple way to reuse sunk training costs by initializing a sparsely activated Mixture-of-Experts model from a dense checkpoint" (arXiv 2212.05055, 9 Dec 2022). **(verified)**

### 1.2 A collision of names, noted once

This repository also uses the acronym **MOE** for the Singapore **Ministry of Education**, in the GovTech-adjacent material on education and public-sector AI. That is a different MOE. Throughout *this* guide, **MoE** means **mixture of experts** and nothing else; where a source is about the Ministry, it is not cited here.

### 1.3 The thesis

The thesis has one sentence and it is the closing sentence of this guide: **every fine-tuning run on a sparse model is also a routing run.** The argument, developed in full in Section 4, is that a fine-tune of an MoE touches two entangled systems — the adapted weights *and* the router that decides which weights see which tokens — and that the second is the one conventional fine-tuning practice has no instrument for. A run can fail because the parameters did not learn, or it can fail because the router never sent the domain's tokens to the experts that were adapted; a run can pass its evaluation and still have collapsed onto a handful of experts, with the cost pushed onto serving-time throughput rather than accuracy. The purpose of this guide is to make the second system visible. It is not a claim that MoE fine-tuning is broken. It is a claim that the MoE-specific failure mode is under-instrumented, and that the instrumentation is cheap if you know to add it.

### 1.4 The boundary, declared by name

This guide owns exactly one subject: **what changes when the model being fine-tuned is sparse.** Four existing guides own everything around it. Their mechanics are written below in one sentence each, with a cross-reference; they are never re-derived here, and if this guide appears to disagree with them, they win on their own subject.

| Owner guide | What it owns | How this guide uses it |
|---|---|---|
| `technology/ai_llm/fine_tuning_frameworks_comparison_guide.md` | What fine-tuning is; full vs PEFT; the PEFT method deep dive (LoRA, QLoRA, DoRA, AdaLoRA, (IA)³); framework comparison; RLHF/alignment; quantisation-aware fine-tuning; data preparation; infrastructure; evaluation; fine-tune vs RAG vs prompting; framework selection. | Section 5 takes its methods as given and asks only *which modules* of a sparse model they should target. |
| `technology/ai_llm/llm_instruction_tuning_guide.md` | SFT mechanics, chat templates and loss masking; SFT vs RLHF vs DPO; PEFT for instruction tuning; IFEval; data curation, decontamination and iteration; a banking-assistant worked example. | Section 5 and Section 7 inherit its data and SFT practice; the MoE layer sits on top. |
| `technology/ai_llm/llm_compression_alternatives_guide.md` | Quantisation frameworks; pruning and sparsity; distillation; model merging; formats; benchmarks; emerging trends including "MoE Compression" (expert pruning, expert quantisation, expert merging, load-balancing-aware quantisation) and QLoRA. | Section 6 and Section 11 lean on its MoE-compression findings; this guide does not repeat its compression taxonomy. |
| `technology/on_prem_llm_deployment_guide.md` | "MoE vs Dense Architectures" (§9); model licensing for banking (§10); quantisation for on-prem (§11); "Fine-Tuning for On-Prem" (§12); the serving stack (§13); serving patterns (§14); air-gapped deployment (§18); governance/audit trail (§23); pitfalls (§25). | Section 8 and Section 12 cross-reference its serving, licensing and fine-tuning-stack sections rather than restating them. |

There is a narrow, deliberate overlap with the on-prem guide's §9, "MoE vs Dense Architectures". That section compares the two *architectures* for serving — memory, compute, throughput, serving complexity. This guide does not re-run that comparison. It assumes the reader has chosen a sparse model, or is considering one, and takes the architecture as the given; the question here begins *after* the model is chosen, at the point a fine-tuning run starts and something about routing decides the outcome.

---
## 2. What a Mixture of Experts Is — The Mechanism This Repository Was Missing

### 2.1 Experts replace the feed-forward block

A dense transformer layer has one attention block and one feed-forward network. An MoE layer keeps the attention block and replaces the feed-forward network with several — the *experts* — plus a small network that chooses among them. Mixtral states the construction plainly: it "has the same architecture as Mistral 7B, with the difference that each layer is composed of 8 feedforward blocks (i.e. experts). For every token, at each layer, a router network selects two experts to process the current state and combine their outputs" (arXiv 2401.04088, *Mixtral of Experts*, 8 Jan 2024). **(verified)**

The idea is older than the current model generation. The sparsely-gated MoE layer was introduced as "up to thousands of feed-forward sub-networks" applied "convolutionally between stacked LSTM layers", reaching "up to 137 billion parameters" (arXiv 1701.06538, 23 Jan 2017). **(verified)** Everything since is a refinement of where the experts live (inside the transformer FFN), how many there are, and how the router is trained.

### 2.2 The learned router and its softmax

The router is not a heuristic; it is trained. It computes, for each token, a score per expert and turns that into a probability distribution over experts, conventionally with a softmax, then keeps the top-k. In the original layer "a trainable gating network determines a sparse combination of these experts to use for each example" (1701.06538). **(verified)** Switch Transformers simplified the choice to a single expert: "Each token is routed to the expert with the highest router probability" (arXiv 2101.03961, full text). **(verified)**

Two facts about the router carry all the way into fine-tuning. First, it is a *parameterised, trainable* module, so it can be trained, adapted, or frozen. Second, its output is a *distribution over a data-dependent set*, so the set of experts a token reaches is a function of the weights and the input, and it can shift without any of the expert parameters being touched.

### 2.3 Top-k sparse activation

Sparsity is what makes the design economical. A token is processed by k of N experts, so the compute per token is roughly k/N of the expert FLOPs while the parameter count is the sum over all N. Mixtral makes the accounting concrete: "Even though each token only sees two experts, the selected experts can be different at each timestep. As a result, each token has access to 47B parameters, but only uses 13B active parameters during inference" (arXiv 2401.04088). **(verified)** The abstract's figure is 47B; the body of the same paper computes 46.7B — the guide cites the abstract's wording and records the discrepancy rather than silently picking one. DeepSeek-V3 is the same shape at larger scale: "671B total parameters with 37B activated for each token" (arXiv 2412.19437). **(verified)**

The name for this in the literature is *conditional computation*: capacity decoupled from cost. The original paper frames it exactly that way — "conditional computation, where parts of the network are active on a per-example basis … increasing model capacity without a proportional increase in computation" (1701.06538). **(verified)**

### 2.4 Shared experts and fine-grained experts

Two refinements changed what "an expert" means. The first is **fine-grained segmentation**: instead of N experts activating k, split each expert into smaller pieces so there are mN and activate mK — a more flexible combination per token. The second is the **shared expert**: reserve some experts that every token always uses, so that the routed experts can specialise without also having to carry the common function. DeepSeekMoE names both: "(1) finely segmenting the experts into $mN$ ones and activating $mK$ from them … (2) isolating $K_s$ experts as shared ones" (arXiv 2401.06066, 11 Jan 2024). **(verified)** The design shipped: DeepSeek-V3's published configuration carries `n_routed_experts: 256` and `n_shared_experts: 1` (deepseek-ai/DeepSeek-V3 `config.json`, retrieved 2026-10-14). **(verified)** Qwen1.5-MoE-A2.7B carries a shared-expert width in its shipped config, `shared_expert_intermediate_size: 5632` alongside `num_experts: 60` (Qwen/Qwen1.5-MoE-A2.7B `config.json`, retrieved 2026-10-14). **(verified)**

The distinction matters for fine-tuning because a shared expert is, by construction, in the tunable path of *every* token; a routed expert is in the path of only the tokens the router sends to it. Adapting a shared expert is closer to dense fine-tuning than adapting a routed one.

### 2.5 Routing variants, including expert-choice routing

The conventional router lets *tokens choose experts* by top-k on a per-token score. Expert-choice routing inverts the axis: "Instead of letting tokens select the top-k experts, we have experts selecting the top-k tokens. As a result, each token can be routed to a variable number of experts and each expert can have a fixed bucket size" (arXiv 2202.09368, 18 Feb 2022; v2 14 Oct 2022). **(verified)** That single change has a large fine-tuning consequence: under expert-choice routing, every expert in a layer receives exactly its bucket of tokens by design, so the "router starved my experts of gradient" failure mode largely disappears — at the cost that a token may receive zero or many experts, and the per-token compute is no longer constant. A reader fine-tuning an expert-choice model is fine-tuning an architecture with different pathology, not a worse one.

A third axis is **auxiliary-loss-free balancing**, in which balance is maintained by an adaptive bias term on the router rather than by an added loss. DeepSeek-V3 "pioneers an auxiliary-loss-free strategy for load balancing" (arXiv 2412.19437). **(verified)** The published config reflects it: the DeepSeek-V3 `config.json` carries the routing fields but **no** `router_aux_loss_coef` key at all, where a Mixtral or Qwen config carries one (deepseek-ai/DeepSeek-V3 `config.json`, retrieved 2026-10-14). **(verified)** This matters later: a fine-tuning recipe that assumes it can inspect or tune an auxiliary-loss coefficient has nothing to inspect on an auxiliary-loss-free model.

### 2.6 The mechanism in one table

| Concept | Where it is defined, with date and kind | What it does |
|---|---|---|
| Sparsely-gated MoE layer | arXiv 1701.06538, 23 Jan 2017 (paper) — up to 137B params, trainable gate | Origin: experts + a learned gate |
| Auxiliary load-balancing loss | arXiv 2006.16668, 30 Jun 2020 (paper) — $\ell_{aux}$ with multiplier $k$ | Pushes the router toward even expert use |
| Top-1 routing | arXiv 2101.03961, 11 Jan 2021 (paper) — highest router probability | One expert per token; fewer communication costs |
| Capacity factor and token dropping | arXiv 2101.03961 (full text) — (total/experts) × capacity_factor | Ceilings per expert; overflow tokens dropped |
| Expert-choice routing | arXiv 2202.09368, 18 Feb 2022 (paper) — experts pick top-k tokens | Fixed bucket per expert; variable per-token experts |
| Shared + fine-grained experts | arXiv 2401.06066, 11 Jan 2024 (paper) — mN/mK, $K_s$ shared | Common knowledge isolated; flexible combinations |
| Auxiliary-loss-free balancing | arXiv 2412.19437, 27 Dec 2024 (paper) — bias term | Balances without an added loss (no aux-loss coef in config) |
| Sparse upcycling | arXiv 2212.05055, 9 Dec 2022 (paper) — dense checkpoint init | Turns a dense model into a sparse one cheaply |

---
## 3. The Load-Balancing Problem

### 3.1 Why a router collapses onto a few experts

A router that minimises its own training objective has no intrinsic reason to spread work. If a few experts happen to become slightly better early, the router's gradient rewards sending more tokens to them; they improve further; the feedback tightens. The result is a router that uses a handful of experts for most tokens and lets the rest idle. This is a *training* pathology with a *serving* consequence, and it is the direct motivation for the auxiliary loss. GShard, which introduced the mechanism at scale, describes the failure it prevents: a poorly balanced router causes "capacity overflow for only a few experts and under-utilization for the remaining ones", and the fix is "an auxiliary loss term $\ell_{aux}$ … added to the overall loss function … with a constant multiplier $k$" (arXiv 2006.16668, 30 Jun 2020; full text). **(verified)**

The expert-choice paper names the same failure from the other direction: "a poor expert routing strategy (e.g. one resulting in load imbalance) can cause certain experts to be under-trained, leading to an expert being under or over-specialized" (arXiv 2202.09368, 18 Feb 2022). **(verified)** Read carefully, that sentence is a fine-tuning warning as much as a pre-training one: *under-trained experts* is what a starved expert is.

### 3.2 The auxiliary balancing loss

The auxiliary loss is a differentiable penalty that rises when the router's average assignment distribution departs from uniform. Switch Transformers gives it a compact form: "the auxiliary loss is computed as the scaled dot-product between vectors $f$ and $P$", where $f$ is the fraction of tokens routed to each expert and $P$ is the fraction of router probability allocated to each expert, scaled by the number of experts and a coefficient. It "is added to the total model loss during training" (arXiv 2101.03961, full text). **(verified)** In practice the coefficient is a config value and it varies widely between models — which is the point of the next subsection.

| Model (published `config.json`, retrieved 2026-10-14) | `router_aux_loss_coef` | Kind |
|---|---|---|
| mistralai/Mixtral-8x7B-v0.1 | 0.02 | configuration **(verified)** |
| mistralai/Mixtral-8x22B-v0.1 | 0.001 | configuration **(verified)** |
| Qwen/Qwen3-235B-A22B | 0.001 | configuration **(verified)** |
| allenai/OLMoE-1B-7B-0924 | 0.01 | configuration **(verified)** |
| microsoft/Phi-3.5-MoE-instruct | 0.0 | configuration **(verified)** |
| openai/gpt-oss-120b | 0.9 | configuration **(verified)** |
| deepseek-ai/DeepSeek-V3 | *(absent — auxiliary-loss-free)* | configuration **(verified)** |

Two entries deserve to be read, not editorialised. Phi-3.5-MoE sets the coefficient to `0.0` and instead carries `router_jitter_noise: 0.01` — balance is sought through noise during training rather than through an added loss term (microsoft/Phi-3.5-MoE-instruct `config.json`, retrieved 2026-10-14). **(verified)** gpt-oss-120b sets the coefficient to `0.9`, an order of magnitude above the Mixtral and Qwen values; the guide reports the config value and does not speculate on the reason (openai/gpt-oss-120b `config.json`, retrieved 2026-10-14). **(verified)** The takeaway for a fine-tuner is narrow and useful: **the balancing mechanism is a property of the checkpoint, and there are at least three distinct mechanisms in the open-model set** — an auxiliary loss, router jitter noise, and an auxiliary-loss-free adaptive bias. A fine-tuning recipe that only knows about the first will misread the others.

### 3.3 The capacity factor and token dropping

Even a balanced router meets a hard ceiling. Each expert is allotted a fixed capacity — Switch Transformers: "(total_tokens / num_experts) × capacity_factor" tokens — and tokens that arrive after the ceiling is full "are dropped" (arXiv 2101.03961, full text). **(verified)** Switch reports the drop rate as typically small, "<1%", when the balancing loss is used with "a high enough coefficient" (arXiv 2101.03961, full text). **(verified)** DeepSpeed exposes the knob as `--moe-train-capacity-factor`, `--moe-eval-capacity-factor` and `--moe-min-capacity`, with the doc noting that "larger numbers could lead to better convergence, but would also lead to slower training" (DeepSpeed-MoE tutorial, retrieved 2026-10-14). **(vendor claim)**

Token dropping is a *training-time approximation* that matters more, not less, once the data distribution shifts. Under a domain fine-tune the router's behaviour can change — a new domain's tokens may concentrate on a subset of experts — and a capacity factor tuned for balanced pre-training data may begin dropping a larger share of the new domain's tokens. Dropped tokens contribute no gradient to the experts they were headed for, so the fine-tune degrades in a way that looks, from the loss curve, like ordinary noise.

### 3.4 What imbalance does to serving, not to training

The reason load balancing matters past the loss is that it is a *throughput* property, not an accuracy property. Experts are distributed across devices by expert parallelism — vLLM's Expert Parallel Deployment documentation explains that EP "allows experts … to be deployed on separate GPUs, increasing locality, efficiency, and throughput" (vLLM docs, retrieved 2026-10-14). **(verified)** If the router sends a disproportionate share of tokens to one expert, that expert's device becomes the bottleneck while the others idle; a layer is only as fast as its slowest expert's queue, and the all-to-all dispatch has to move tokens to wherever the hot expert lives.

| Imbalance symptom | Training effect | Serving effect |
|---|---|---|
| A few experts take most tokens | Some experts under-trained (arXiv 2202.09368) | The hot expert's device saturates; EP lanes idle |
| Router probability concentrates | Aux loss rises; if untuned, collapse | Uneven all-to-all; throughput bounded by one expert |
| Capacity overflow | Overflowing tokens dropped (arXiv 2101.03961) | Dropped/rerouted tokens change decode behaviour |

This is why "it worked in evaluation" is an incomplete verdict for a sparse model, a claim Section 9 develops: an accuracy number measures the tokens that *did* get processed correctly, and says nothing about whether the attention of the expert set was spread across devices or concentrated on one. A fine-tune can leave task accuracy untouched and quietly make the model much more expensive to serve, because the routing moved.

---
## 4. Why Fine-Tuning a Sparse Model Is Not Fine-Tuning a Dense One

### 4.1 The claim

Fine-tuning a dense model optimises one set of weights against one loss. Fine-tuning a sparse model optimises *and* routes: the adapted weights are what the loss moves, and the router is what decides which of those weights ever contribute to which token. The two are coupled through the forward pass and decoupled in almost every fine-tuning instrument. This is the load-bearing claim of the guide, and it can be stated as a single equality that conventional practice never checks:

> the fine-tune that actually happened = (the experts this data reached) ∩ (the experts this task needs).

If that intersection is empty — if the domain's tokens all route to experts the recipe did not adapt — the fine-tune did nothing for the domain, however well the loss curve behaved. If that intersection is everything — if the adapted experts are the ones serving every token — the fine-tune is effectively dense and the sparsity bought nothing. Neither failure is visible in a loss curve.

### 4.2 Only the selected experts receive gradient from a token

A token's gradient flows only through the experts the router selected for it. This is not a subtlety; it is the definition of sparsity. If a token is routed to 2 of 64 experts, then 62 experts receive exactly zero gradient from that token. Over a dataset, the *tunable set is data-dependent*: the subset of experts that receives any gradient at all is determined by where the router sends the data, not by the fine-tuning configuration alone. This is the mechanism behind the empirical finding that per-layer routing is skewed: "a small subset of experts handles most tokens in each layer, while many others are rarely activated ('cold')" (MoE-Sieve, arXiv 2603.24044, 25 Mar 2026). **(verified)** Most LoRA fine-tuning of an MoE "applies adapters to every expert" — but of the adapters that receive gradient, only a fraction are ever reached by the data, and the "cold" ones receive gradient noise at best.

There is a second, sharper version of the same point, from the negative-transfer literature: "Although these domains show near-disjoint expert routing, adding biomedical data substantially increases code perplexity, indicating that routing separation alone may not prevent negative transfer" (arXiv 2609.03150, 2 Sep 2026). **(verified)** Even when routing *did* separate the domains cleanly, the adapters interfered — because the interference lived "mostly from nearly orthogonal domain gradients competing within the same low-rank adapter subspace". The lesson is double-edged: routing separation is necessary but not sufficient, and the adapter's own capacity is a second site of contention.

### 4.3 The router is itself trained — or frozen, and that is a decision

The router is a trainable module. In a LoRA fine-tune the router is usually *not* in the LoRA target set, which means that unless it is explicitly targeted, it receives no gradient from LoRA (LoRA touches only the adapted weights, and the router's own parameters are frozen by omission). But under *full* fine-tuning, or when the router is included in the tunable set, it does train — and then the routing itself moves during the fine-tune. The design space is therefore three-way, not two-way:

| Router treatment | What it means | What it costs |
|---|---|---|
| Frozen (implicit — not in the tunable set) | Routing distribution is fixed at the pretrained state; only experts/attention move | No router drift; but a domain the router never routed well cannot be helped — the router cannot learn to send domain tokens to the adapted experts |
| Frozen (explicit, enforced) | As above, made deliberate so a bug or a merged adapter cannot move it | Same; also the design SpawnLoRA chose on purpose (arXiv 2609.03150) **(verified)** |
| Trained (router in the tunable set) | Router learns the domain; new routing can help and can hurt | Chance of collapse, of catastrophic interference, and of load imbalance — the thing Section 3 is about |

The choice is not obvious. A frozen router protects against collapse but forbids the very adaptation that a far-from-pretraining domain may need. A trained router can find the domain but can also drift into imbalance. The literature does not settle it: SpawnLoRA "keeping the router fixed" and solving the interference in the adapters (arXiv 2609.03150) is one answer; DR-LoRA using "routing frequency and gradient-based rank importance" to grow the *right* experts' adapters (arXiv 2601.04823, 8 Jan 2026) implicitly lets routing inform adaptation. **(verified)**

### 4.4 A domain fine-tune can fail because the router never sent the tokens to the right experts

This is the failure this guide exists to name. The parameters can learn perfectly and the fine-tune still not work, because the router — frozen or drifted — sends the domain's tokens to experts that were not adapted. Symptoms: a fine-tune whose loss falls but whose downstream accuracy does not move; a task that "did not respond to fine-tuning" that a dense model of the same size responded to. The diagnosis is not "the learning rate was wrong"; it is "check where the domain's tokens went, and check whether the adapted experts are the ones they went to". MoE-Sieve operationalises exactly this: profile routing on a calibration set, then adapt only the experts that the domain actually reaches — "tuning only the top 25% routed experts per layer remains competitive with full LoRA, with mean differences within +/-1 percentage point across all conditions" while cutting trainable parameters by "70-73%" (arXiv 2603.24044). **(verified)** The corollary is uncomfortable: if adapting the top-25% routed experts matches full LoRA, then adapting the *other 75%* — which uniform LoRA does — is largely wasted, and the wasted adapters contribute the "gradient noise without improving accuracy" that MoE-Sieve attributes to cold experts.

### 4.5 A run can succeed on the evaluation set while load balancing has collapsed

Accuracy and balance are different variables. A fine-tune can hold task accuracy and still leave routing concentrated on a few experts — and because the bottleneck is throughput, not correctness, the regression shows up at serving, not on the eval set. The pathological version is a router that specialises the *new domain* onto a small set of experts while starving the others of that domain's traffic; the model answers the domain well and serves it badly, with one device carrying the load. There is no accuracy metric that catches this, and no loss curve that shows it. It is visible only in a routing histogram — Section 9.

### 4.6 What "targeting the experts" means

"Target the experts" is not one decision but three, and they are routinely conflated:

- **Attention-only adaptation.** LoRA on `q_proj`, `k_proj`, `v_proj`, `o_proj` and nothing else — the default in most PEFT recipes, because those are `nn.Linear` modules that LoRA handles without special machinery. This trains *no* expert weights. It can work, briefly, for a narrow style task; for a domain shift it is the "adapting only the attention modules and leaving the experts untouched" failure mode of Section 10.
- **Expert-module adaptation.** LoRA on the experts themselves. In current HF Transformers this frequently means targeting `nn.Parameter` tensors rather than modules, which needs PEFT's `target_parameters` (Section 5). This is where the sparse gradient question bites: each expert adapter is only trained by the tokens routed to it.
- **Router adaptation.** LoRA or full training on the gate. This is the decision of §4.3, and it is the one most recipes make by accident — by leaving the router out of the target set and not noticing.

### 4.7 What freezing the router does, and costs

Freezing the router is the conservative default and it is often right. It guarantees no router drift and no new collapse; it keeps the routing distribution at the pretrained state, which was trained on a large, balanced corpus and is usually far more stable than a domain fine-tune could make it. What it costs is adaptation of the *decision*: the pretrained router was trained to route general text, and a genuinely new domain may be routed by it to the wrong experts. Freezing makes that permanent. The honest formulation is: freezing the router trades a small risk of collapse for a small risk of a domain the router never learns to serve. Which risk dominates depends on how far the domain is from pretraining — the same variable that governs the counter-case in §4.9.

### 4.8 Why "it worked in evaluation" is an incomplete verdict

For a dense model, a held-out accuracy number is close to a sufficient verdict: the parameters are what they are, and if accuracy holds, the model is as good as its evaluation says. For a sparse model the verdict is incomplete for three reasons. First, evaluation measures the tokens that were processed correctly, not *which experts processed them* — a collapsed router and a healthy router can post the same accuracy. Second, a sparse model carries a second cost axis — serving throughput — that a task metric does not touch, and load collapse moves that axis. Third, sparsity adds a stability dimension the literature flags explicitly: ST-MoE exists as "a design guide" precisely because progress was "hindered by training instabilities and uncertain quality during fine-tuning" (arXiv 2202.08906, 17 Feb 2022). **(verified)** ST-MoE is a fine-tuning-time warning from the sparse-model literature itself, and it says the quality of a sparse fine-tune is harder to predict, not easier.

### 4.9 The counter-case, stated honestly

For many small fine-tunes the distinction is theoretical. If the domain is close to the pretraining distribution, the pretrained router already routes its tokens sensibly; a few hundred steps of LoRA move neither the routing nor the balance appreciably; the sparse model behaves like a dense one and the MoE-specific instrumentation adds cost for no finding. That is a real and common case, and a guide that pretended otherwise would be selling ceremony. The conditions that make the distinction *practical* — that make §4.2–§4.5 the explanation of what you are seeing — are:

- **The domain is far from the pretraining distribution**, so the pretrained router has no reason to route it well.
- **The domain is long-tail or low-resource**, so it exercises few experts and drives little gradient into the rest.
- **There are many-task adapters**, so several adapters contend inside the same expert subspace — the SpawnLoRA finding.
- **The router is in the tuned set**, so routing can move (and collapse).
- **The learning rate is high**, so both experts and router can move far in few steps.
- **There are many steps**, so small per-step routing drift accumulates into a different distribution.

When several of these hold at once, the routing is no longer a background constant and the guide's core claim stops being theoretical. When none holds, a reader is entitled to run the dense recipe and add the routing checks only if the result surprises them.

---
## 5. The Method Choice, Cross-Referenced Not Re-Derived

### 5.1 What this section does not do

It does not restate what LoRA, QLoRA or full fine-tuning are. Those are owned: the PEFT method deep dive (LoRA, QLoRA, DoRA, AdaLoRA, (IA)³) is §4 of `technology/ai_llm/fine_tuning_frameworks_comparison_guide.md`; the choice between full and PEFT is its §3; quantisation-aware fine-tuning is its §7; the SFT-side use of LoRA is §6 of `technology/ai_llm/llm_instruction_tuning_guide.md`; the on-prem LoRA/QLoRA-versus-full decision is §12.1 of `technology/on_prem_llm_deployment_guide.md`. Where those guides define a method, this guide takes the definition as given and asks only the sparse-specific question: **which modules of a sparse model should the method target?**

### 5.2 Which modules get adapted in a sparse model

A dense fine-tuning recipe targets a flat list of modules — typically the attention projections, sometimes the MLP projections, occasionally the embeddings. A sparse model adds two categories that a dense recipe has no language for: the **experts** and the **router**.

The transformer machinery people adapt is the same list the sibling guides give — `q_proj`, `k_proj`, `v_proj`, `o_proj` for attention, and the gate/up/down projections of the MLP for the feed-forward path. But in many MoE models in HF Transformers "instead of using `nn.Linear`, an `nn.Parameter` is used" for the expert weights — so targeting them needs PEFT's `target_parameters` rather than `target_modules`, because "PEFT normally overwrites the `forward` method for LoRA, but for `nn.Parameter`, there is none" (PEFT `package_reference/lora.md`, retrieved 2026-10-14). **(verified)** The documented names are model-specific:

| Model family | Expert parameters to target | Source (kind, date) |
|---|---|---|
| Llama-4 | `feed_forward.experts.gate_up_proj`, `feed_forward.experts.down_proj` | PEFT `lora.md` **(verified)** 2026-10-14 |
| Mixtral / Qwen3-MoE / GPT-OSS | `mlp.experts.gate_up_proj`, `mlp.experts.down_proj` | PEFT `lora.md` **(verified)** 2026-10-14 |

PEFT's caveat is worth reading twice for a sparse model: the framework "allows to target 2-dim or 3-dim `nn.Parameter`s", and "in the case of a 3-dim parameter … the 0th dimension is the expert dimension" (PEFT `lora.md`, retrieved 2026-10-14). **(verified)** That 3-D tensor is a *fused* stack of every expert's weights in one parameter; it is why a plain `r=64` LoRA across the whole expert stack is a very different budget from `r=64` on a dense linear layer. PEFT's own recipe multiplies the per-layer rank down by the expert count — `effective_r = max(1, r // num_experts)` via `rank_pattern` — "to keep the total LoRA parameter budget similar to dense layers" (PEFT `lora.md`). **(verified)**

### 5.3 The router in the target set — the freeze-or-train decision

The method choice forces a decision that a dense recipe never faces: is the router a trainable module? Three positions, developed in §4.3, and the method maps onto them:

- **LoRA (adapters) targeting only attention and/or experts, router not in the target set.** The router is frozen by omission. This is the common default and the one most recipes carry without deciding it.
- **Full fine-tuning, or LoRA explicitly extended to the gate.** The router trains. This is the case where routing can move, for good and ill, and where §4.3's costs are live.
- **LoRA on experts with the router explicitly and intentionally frozen.** The deliberate version of the first case — the position SpawnLoRA takes: "keeping the router fixed" while addressing contention inside the experts (arXiv 2609.03150). **(verified)**

LoRA is the natural instrument when the goal is to hold routing still: adapters on a frozen base leave the router at its pretrained parameters unless the recipe deliberately adds the gate. That is a feature — it makes the freeze explicit and cheap to verify — and a limitation, because it forbids the router from learning to serve a new domain. The decision is a *domain-distance* decision, and §4.9 gives the conditions under which it matters.

### 5.4 QLoRA and the merging step

QLoRA (quantise the base, train LoRA on top) is owned by the comparison guide's §4.2 and §7.1 and the compression guide's §12.6; this guide adds only the sparse wrinkle. On a sparse model the base is large *in total* parameters even though it is small *in active* parameters, so the memory question is different from a dense model of the same active size: the whole expert stack must be resident, quantised or not, which is the same memory-per-total-parameter fact the on-prem guide's §9 and §11 describe for serving. The instruments are the same; the size you are quantising is the total, not the active.

The step that QLoRA and LoRA both need, and that is a genuine MoE-specific hazard, is **merging**. PEFT's documentation is explicit: "when targeting expert parameters, PEFT can add a substantial runtime overhead. The reason is that PEFT always materializes the LoRA contribution for _each expert_ even if only a small amount of experts is required." During training this is tolerable, but at inference, with a KV cache, "only a small amount of experts is activated", so "using LoRA on MoE layers can result in a substantial slowdown at inference time. The recommendation is thus to merge the weights (`model.merge_adapter()` or `model = model.merge_and_unload()`)" (PEFT `lora.md`, retrieved 2026-10-14). **(verified)** In other words: on a sparse model, leaving adapters unmerged is not merely slower, it defeats the sparsity — every expert's adapter contribution gets materialised whether or not that expert is routed to. Section 6 continues the merge-and-restore question.

---
## 6. Targeting the Experts

### 6.1 Which experts to adapt

The default — adapt every expert — is defensible and, on the evidence, also wasteful. MoE-Sieve profiled routing "across architectures and tasks" and found it "highly skewed", then showed that adapting only the top 25% most-routed experts per layer "remains competitive with full LoRA, with mean differences within +/-1 percentage point across all conditions", while reducing "LoRA trainable parameters by 70-73%" and "wall-clock training time by up to 50%" (arXiv 2603.24044, 25 Mar 2026). **(verified)** Two control findings sharpen it. Random expert selection at a matched budget was "about 2.5 percentage points worse" than routing-guided selection — so the *routing signal*, not merely the smaller budget, is doing the work. And greedy per-layer budget optimisation "does not improve over uniform top-k" — so the simple rule (top-25% per layer) is as good as a cleverer allocation. DR-LoRA generalises the "adapt every expert uniformly" critique: uniform rank "ignores the heterogeneous specialization of pretrained experts", leaving "task-relevant experts under-provisioned, while less relevant ones receive redundant parameters", and instead grows rank on the experts with high "routing frequency and gradient-based rank importance" (arXiv 2601.04823, 8 Jan 2026; v6 31 Aug 2026 — a preprint, and v6 postdates v1). **(verified)**

The selection question has one instrument: routing. Profile where the domain's tokens go on a calibration set, and adapt the experts that receive them. Section 7 is the procedure.

### 6.2 Is adaptation per-expert, or shared?

There are three structural choices, and they are not equivalent:

- **One adapter per expert.** Each expert gets its own low-rank update. Maximum capacity, maximum parameter count, and the adapters are as sparse in gradient as the experts are in routing — an expert that never receives the domain's tokens never trains its adapter.
- **One shared adapter across the fused expert stack.** A single LoRA over the 3-D expert tensor. Cheaper, but it cannot represent per-expert heterogeneity, which is the exact thing DR-LoRA says uniform allocation gets wrong.
- **A mixture over adapters.** "Mixture of LoRA Experts" applies an MoE over LoRA layers (arXiv 2404.13628, 21 Apr 2024); MoELoRA "considers LoRA as Mixture of Experts (MoE)" and adds contrastive learning "to mitigate the random routing phenomenon" (arXiv 2402.12851, 20 Feb 2024). **(verified)** These fuse the two ideas — a router over *adapters* — and are a different design from adapting the model's own experts; they are noted because they exist, not because they are the default.

PERFT provides the framing for the whole space: a "unified framework for integrating PEFT modules directly into the MoE mechanism", with "design dimensions including various functional and composition strategies", and reports "intriguing dynamics" when adapting OLMoE-1B-7B and Mixtral-8x7B (arXiv 2411.08212, 12 Nov 2024). **(verified)** The word "dynamics" is the honest state of the art: how PEFT interacts with routing is not fully mapped, and a practitioner should expect to measure rather than assume.

### 6.3 Merging and restoring adapted experts

Two operations a sparse fine-tune needs that a dense one does not:

- **Merge.** Folding the adapters back into the base weights removes the per-expert materialisation overhead that PEFT warns about (PEFT `lora.md`, retrieved 2026-10-14). **(verified)** On a sparse model this is not optional for throughput; it is the difference between the sparsity and no sparsity.
- **Restore.** Reverting a subset of expert adapters — the cold experts that received no useful gradient — is a legitimate cleanup. If MoE-Sieve is right that cold experts contribute only noise, restoring them to the base removes the noise cheaply. The instrumentation is the routing count: an expert whose adapter trained on near-zero tokens has no information to contribute.

PEFT adds a constraint for multi-adapter work: "Multiple LoRA adapters … that use `target_parameters` are supported, but all of them must target the same set of parameters" (PEFT `lora.md`). **(verified)** For a bank running several domain adapters off one sparse base, that constraint shapes the design — the adapters must agree on which expert parameters they touch.

### 6.4 Re-balancing after a fine-tune

A fine-tune can move the routing, and after it moves the balance question is live again. Re-balancing means: measure the post-fine-tune routing histogram (Section 9); compare it against the pre-fine-tune histogram; and if it has concentrated, decide whether to (a) accept it and plan the serving capacity for the hot expert, (b) re-run with a smaller router perturbation — a lower learning rate, fewer steps, or the router frozen, or (c) if the router trained and the model is auxiliary-loss-free, accept that there was never an aux coefficient to tune and the balance was always maintained by the adaptive bias you did not see. The re-balancing question has no single answer, but it has a single precondition: you cannot re-balance what you have not measured.

---
## 7. The Data Question, in the MoE's Terms

### 7.1 The question a dense dataset review does not ask

The sibling guides own data preparation (comparison guide §8), curation, decontamination and iteration (instruction-tuning guide §8). All of that practice is about the *content* of the data: is it clean, is it on-domain, is it decontaminated against the eval set, is it big enough. The MoE adds a question those guides do not ask and cannot: **does this data, when it runs through this model, actually reach the experts you are going to adapt?** A perfectly curated banking dataset can still route, on the model in hand, to a set of experts your recipe does not touch — and then the curation was necessary and insufficient.

### 7.2 Routing histograms on a calibration set

The instrument is cheap: take a small calibration set — a few hundred to a few thousand representative domain examples — run them forward with `output_router_logits` (or the model's equivalent routing hook), and count, per layer, how many tokens each expert received. The output is a per-layer histogram. Three readings matter:

- **Coverage.** What fraction of experts in each layer saw any tokens from the domain? If a layer sends everything to 6 of 64 experts, only those 6 can be usefully adapted in that layer.
- **Concentration.** Is one expert or a few dominating? A domain that routes 80% of its tokens to two experts is a domain that the pretrained router has already decided is narrow — and a fine-tune of the other experts will not reach it.
- **Drift.** Run the same histogram before and after the fine-tune. The difference is whether the fine-tune *moved* the routing, which is the question §4.5 insists evaluation cannot answer.

MoE-Sieve built its whole method on this histogram: "profile routing counts on a small calibration set, select the top-k most-routed experts per layer, and apply LoRA only to those experts" (arXiv 2603.24044, 25 Mar 2026). **(verified)** The method is simple enough to implement without a library, and the finding it rests on — that the top-25% carries the adaptation — is the reason the profile is worth taking even if you choose to adapt every expert anyway.

### 7.3 Jaccard routing overlap — the second diagnostic

Coverage tells you where the data goes; it does not tell you whether two datasets share experts. The "Routing Is Not Enough" paper introduces "Jaccard routing overlap" to measure exactly that — "expert sharing" between domains — and "adapter-gradient cosine similarity" to measure "update compatibility" (arXiv 2609.03150, 2 Sep 2026). **(verified)** The finding is the guide's sharpest negative result: domains with *near-disjoint routing* still suffered negative transfer, because the interference was in the shared adapter subspace, not in the shared experts. Jaccard routing overlap is therefore a *necessary* check (if two domains route to the same experts, they will contend) but not a *sufficient* one (near-zero overlap does not guarantee no interference). The companion diagnostic — gradient cosine similarity between adapters — is the one that catches the residual case.

### 7.4 What to do with the reading

The data-routing reading maps onto three actions:

| Reading | Interpretation | Action |
|---|---|---|
| Domain routes to few experts, recipe adapts all | Most adapters get no signal | Narrow the target set (MoE-Sieve-style top-k), or accept the waste knowingly |
| Domain routes to experts the recipe does *not* target | The intersection is empty | Re-target, or train the router — or conclude the model is wrong for the domain |
| Two domains route disjointly but interfere | Contention is in the adapter subspace | Jaccard overlap + gradient cosine; consider per-domain sub-adapters |
| Routing shifts a lot pre-to-post fine-tune | The router moved | Check balance (Section 3), serve accordingly (Section 8) |

The single most useful habit this section asks for is to treat the routing histogram as a *first-class dataset artefact* — produced at data-review time, kept next to the dataset, and re-run after every fine-tune — rather than as an optional debugging step reached for only when something breaks.

---
## 8. The Infrastructure

### 8.1 Expert parallelism vs tensor and data parallelism

Sparse models introduce a third axis of parallelism that dense training did not need. **Expert parallelism (EP)** places different experts on different devices; **tensor parallelism (TP)** splits each weight matrix across devices; **data parallelism (DP)** replicates the model across devices and splits the batch. Megatron's definition is precise: EP "distributes experts of a Mixture of Experts (MoE) model across GPUs. Unlike other model-parallel techniques, EP is applied to only the expert layers and does not impact the parallel mapping of the rest of the layers" (Megatron Bridge *Parallelisms Guide*, retrieved 2026-10-14). **(verified)** The MoE layer shards experts; the attention and embedding layers do not.

vLLM's serving documentation makes the interaction concrete: EP is enabled with `--enable-expert-parallel` and "the EP size is automatically calculated as `EP_SIZE = TP_SIZE × DP_SIZE`". With `TP=2, DP=4` (eight GPUs), "expert layers form an EP group of size 8, with experts distributed across all GPUs", while "attention layers use TP=2 within each of the 4 DP groups" (vLLM *Expert Parallel Deployment*, retrieved 2026-10-14). **(verified)** And the reason to do it: with EP enabled, "expert layers switch to expert parallelism, which can provide better efficiency and locality for MoE models" (vLLM, retrieved 2026-10-14). **(verified)**

| Parallelism | Splits what | Applies to | MoE-specific cost |
|---|---|---|---|
| Expert parallel (EP) | The experts, across devices | Expert layers only | All-to-all dispatch of tokens (below) |
| Tensor parallel (TP) | Each weight matrix | All layers | Communication per layer, as for dense |
| Data parallel (DP) | The batch, model replicated | All layers | None specific to MoE; often paired with EP |

### 8.2 The memory profile: total resident, active computed

A sparse model's memory is set by its *total* parameters and its compute by its *active* parameters. DeepSeek-V3 is "671B total parameters with 37B activated for each token" (arXiv 2412.19437, 27 Dec 2024) — so a fine-tune or a serve must hold the 671B (quantised or not) while computing at roughly the 37B rate. **(verified)** This is the same asymmetry the on-prem guide's §9 ("MoE vs Dense Architectures") describes for serving, and it is why "the model is only 37B" is a misleading sizing input: it tells you the compute, not the footprint. For a fine-tune, the implication is that the optimiser state, the adapter tensors and any fused expert parameters are all sized against the total, while the throughput reward comes from the active fraction.

### 8.3 All-to-all dominates, in training and in serving

Because experts live on different devices, every MoE layer must *dispatch* tokens to their experts and *combine* the results — an all-to-all exchange. This is the communication that MoE serving centres on, and it is why the choice of all-to-all backend is a serving decision, not a detail. vLLM exposes it directly with `--all2all-backend`, offering `allgather_reducescatter` (default, general purpose), `deepep_high_throughput` (multi-node prefill), `deepep_low_latency` (multi-node decode), and `flashinfer_nvlink_one_sided` / `_two_sided` for multi-node NVLink (vLLM *Expert Parallel Deployment*, retrieved 2026-10-14). **(verified)**

Crucially for this guide, the all-to-all cost is a function of *routing*. If one expert is hot, more tokens must travel to its device and fewer to its idles, and the exchange becomes uneven — the load-balancing problem of Section 3 surfacing as a communication problem. DeepSpeed-MoE, the training-side counterpart, addresses the same cost with "novel MoE architecture designs and model compression techniques that reduce MoE model size by up to 3.7x" and a training cost "5x saving for auto-regressive language models" (arXiv 2201.05596, 14 Jan 2022; v2 21 Jul 2022). **(verified)** DeepSpeed also provides high-performance MoE token dispatchers under the Megatron Bridge ("DeepEP and HybridEP"), optimised per GPU generation (Megatron Bridge *Parallelisms Guide*, retrieved 2026-10-14). **(verified)** The dispatcher and the routing are two ends of the same wire.

### 8.4 What this section cross-references rather than restates

The serving *stack* — model servers, gateways, the KV-cache and batching machinery, autoscaling, air-gapped deployment — is owned by `technology/on_prem_llm_deployment_guide.md`: §13 (the serving stack architecture), §14 (serving patterns), §6 (the parallelism/config example), §18 (air-gapped deployment), §20 (operations, monitoring, capacity). This guide does not re-derive any of it. What it adds is the sentence those sections need: for an MoE, the serving stack inherits EP as a first-class parallelism mode and the all-to-all as a first-class cost, so a throughput model built from a dense deployment will under-count both. The on-prem guide's §9 already flags the pattern; this guide supplies the fine-tuning-side reasons it holds.

---
## 9. Evaluating a Sparse Model and Its Routing

### 9.1 Two evaluations, not one

The sibling guides own task evaluation: task-specific metrics (comparison guide §10), general-benchmark regression testing (§10.2), IFEval (§7 of the instruction-tuning guide), and safety evaluation. Those apply unchanged to a sparse model — you still need to know the task accuracy. What this section adds is a *second* evaluation, run beside the first: **routing health**. The claim of Section 4 is that an accuracy number is an incomplete verdict for a sparse model; the response is to make the routing measurable and run it every time.

| Axis | What is measured | Instrument | Why it matters for MoE |
|---|---|---|---|
| Task quality | Accuracy / IFEval / task metrics | The sibling guides' evals | Unchanged; measures the tokens processed correctly |
| Expert utilisation | How many experts receive tokens, and how evenly | Per-layer routing histogram on a calibration set | Detects starvation and concentration (Section 3) |
| Routing health after adaptation | Did the fine-tune move the routing? | Histogram pre-vs-post; Jaccard overlap | Catches collapse and interference an accuracy number hides |
| Serving cost | Throughput / latency under routing | The serving stack's own metrics (on-prem §13–14) | Imbalance is a throughput regression, not an accuracy one |

### 9.2 Expert utilisation

Run the domain's calibration set through the fine-tuned model with router logging and count tokens per expert per layer. Read coverage and concentration (§7.2). The two numbers to keep are the fraction of experts that received any tokens and the share taken by the top expert. A healthy sparse model spreads the domain across many experts; a collapsed one concentrates it. The pre-fine-tune histogram is the baseline, and the post-fine-tune histogram is the finding.

### 9.3 Did adaptation collapse the routing?

Collapse has a specific signature: routing concentration rises from pre to post, and usually the experts that concentrate are the ones adapted. The check is a comparison, so it needs the pre-histogram — which is why §7.2 insists the histogram be taken before the run, not after the surprise. If the router was frozen (not in the target set), collapse should be near-impossible and the histogram should be near-identical; a *large* change with a frozen router is itself a finding, because it means something other than the router moved the routing — typically the experts, whose adapted outputs feed back into the gate's input. If the router was trained, a shift is expected and the question is only whether it is healthy.

### 9.4 Why an accuracy number alone does not answer the question

Three independent reasons, restated because they are the section's whole point:

- **Same accuracy, different routing.** A collapsed router and a healthy router can post identical task accuracy; the difference lives on the throughput axis, which no task metric touches.
- **Same accuracy, different cost.** The model that routes domain traffic to one expert serves it at a fraction of the throughput of the model that spreads it — invisible to accuracy.
- **Different accuracy, same loss.** A fine-tune that trains well but routes the domain to untrained experts shows a healthy loss and a flat accuracy; the diagnosis is routing, not learning rate.

ST-MoE is the literature's own warning that sparse quality is harder to predict during fine-tuning — it exists as "a design guide" because progress was "hindered by training instabilities and uncertain quality during fine-tuning" (arXiv 2202.08906, 17 Feb 2022). **(verified)** The practical consequence is that for a sparse model the evaluation set should carry routing diagnostics alongside its task metrics, so that "it worked" means both "the answers were right" and "the routing that produced them is still healthy".

### 9.5 Instrumenting the run

The routing checks are cheap only if they are scheduled, not improvised. A workable cadence for a sparse fine-tune:

| Cadence | What to log | Why |
|---|---|---|
| Before the run | Routing histogram on the domain calibration set; adapted-parameter list; router frozen-or-trained decision | Establishes the baseline and makes the target-set decision explicit (§5, §6, §7) |
| Every N checkpoints | Routing histogram on a fixed calibration sample; expert-utilisation coverage and top-expert share; token-drop rate | Catches collapse and capacity overflow as they happen, not after shipping (§10.1, §10.4) |
| At the end | Post-run histogram against the baseline; Jaccard overlap if multiple domains; serving throughput re-measurement | Answers "is the run finished" and "did the routing move" (§9.3); sizes capacity (§8.3) |
| In the audit trail | The histogram pair, the adapted-expert list, the router decision | The MoE-specific artefact of §12.2 |

The instrument is a forward pass with router logging over a fixed sample; it costs a fraction of a training step and it is the only thing that answers the questions Sections 4, 7 and 9 raise. A run that skips it is not wrong — it is unverified on the axis that makes the model sparse.

---
## 10. The Failure Modes

Each failure is given as symptom, cause and guardrail. The guardrails are cheap by design; the failure modes are expensive by nature.

### 10.1 Router collapse

- **Symptom.** A small set of experts handles most tokens; the post-fine-tune routing histogram is far more concentrated than the pre-fine-tune one; serving throughput falls on the hot expert's device.
- **Cause.** A router allowed to train drifts toward the experts that win early; the auxiliary loss (or jitter noise, or adaptive bias) that balanced pre-training was either absent, mis-scaled, or overridden by the fine-tune objective.
- **Guardrail.** Take the pre-fine-tune histogram; freeze the router unless there is a domain reason not to; if the router must train, keep the balance mechanism the checkpoint shipped and watch the histogram each checkpoint.

### 10.2 Catastrophic interference in the router

- **Symptom.** The domain gets better and an unrelated capability regresses; or, per the SpawnLoRA finding, a near-disjoint domain still damages another domain because the adapters share a low-rank subspace.
- **Cause.** Near-orthogonal gradients from different domains competing inside the same adapter subspace, even when routing separates the tokens (arXiv 2609.03150). **(verified)**
- **Guardrail.** Measure Jaccard routing overlap and adapter-gradient cosine similarity between domains; when overlap is low but interference appears, separate the adapters structurally rather than relying on routing to separate the updates.

### 10.3 Adapting only the attention modules, leaving the experts untouched

- **Symptom.** A fine-tune that improves style or format but not domain knowledge; the experts — where the feed-forward capacity lives — were never in the tunable set.
- **Cause.** The default PEFT recipe targets `q_proj`/`k_proj`/`v_proj`/`o_proj` because they are `nn.Linear` modules; the experts are `nn.Parameter` tensors that need explicit targeting (PEFT `lora.md`, retrieved 2026-10-14). **(verified)**
- **Guardrail.** Check the adapted-parameter list against the model's actual module names; if no expert parameter appears, the experts are frozen and the fine-tune is an attention-only adaptation.

### 10.4 Token dropping under the capacity factor

- **Symptom.** A fine-tune that converges but underperforms on a domain whose routing became concentrated; some domain tokens contribute no gradient.
- **Cause.** Each expert has a capacity ceiling of (total_tokens / num_experts) × capacity_factor, and "tokens … are dropped" on overflow (arXiv 2101.03961, full text). **(verified)** A capacity tuned for balanced pre-training data can overflow once a domain concentrates the routing.
- **Guardrail.** Watch the drop rate during fine-tuning; if it rises, raise the capacity factor (at a throughput cost, per DeepSpeed's `--moe-train-capacity-factor`) or re-balance the routing so no expert overflows.

### 10.5 A successful evaluation beside a serving-time regression

- **Symptom.** Task accuracy holds or improves; latency or cost per query rises after deployment; the throughput model built for the dense baseline no longer predicts the MoE.
- **Cause.** The fine-tune concentrated routing on a few experts, so the all-to-all and the per-device queues are now uneven, even though the answers are right (Section 3.4, Section 8.3).
- **Guardrail.** Evaluate routing health beside task accuracy (Section 9); re-measure serving throughput post-fine-tune rather than inheriting the pre-fine-tune baseline; plan capacity for the observed concentration.

### 10.6 The upcycle-then-fine-tune mistake

- **Symptom.** A dense model was upcycled into a sparse one (or a sparse model trained from a dense checkpoint) and then fine-tuned, and the result behaves worse than either the dense original or the expected sparse model.
- **Cause.** Upcycling "initializ[es] a sparsely activated Mixture-of-Experts model from a dense checkpoint" (arXiv 2212.05055, 9 Dec 2022) and then requires its own *post-upcycle training* before it is a usable sparse model — the upcycled experts start as copies of the dense FFN, and until they diverge the routing has nothing to distinguish. **(verified)** Fine-tuning an un-diverged upcycle asks adapters to specialise experts the router cannot yet tell apart.
- **Guardrail.** Treat upcycle training and downstream fine-tuning as two stages, not one; confirm the experts have diverged (routing is informative and balanced) before starting the domain fine-tune.

### 10.7 The failure-mode table

| # | Failure | Primary symptom | Cheapest guardrail |
|---|---|---|---|
| 1 | Router collapse | Concentrated histogram; hot-expert throttling | Pre/post histogram; freeze router by default |
| 2 | Adapter-subspace interference | One domain regresses another | Jaccard overlap + gradient cosine |
| 3 | Attention-only adaptation | Style improves, knowledge does not | Audit the tuned-parameter list |
| 4 | Capacity overflow | Domain tokens dropped | Watch drop rate; raise capacity or re-balance |
| 5 | Eval passes, serving regresses | Latency/cost up, accuracy flat | Re-measure routing + throughput post-run |
| 6 | Upcycle then fine-tune | Worse than expected sparse model | Two-stage: upcycle-train, then fine-tune |

---
## 11. The Open Models, Factually

This section lists open-weight sparse models a reader might actually fine-tune, with the figures taken from the published `config.json` (kind: configuration) or the published model card (kind: model card), each retrieved 2026-10-14, and the licence exactly as the repository publishes it. **There is no quality ranking here.** A parameter count is not a recommendation; the ordering is by scale, not by merit. Where the model card gives total/active parameters that the `config.json` does not, the card is cited as the source. No benchmark score appears — none was retrieved from a primary source, and the hazard rule forbids printing one from memory.

| Model | Total : active params | Experts / experts-per-token | Layer / other routing figures | Context | Licence (as published) | Source kind |
|---|---|---|---|---|---|---|
| Qwen1.5-MoE-A2.7B | — | 60 experts, 4 per token | `moe_intermediate_size` 1408, **shared expert** `shared_expert_intermediate_size` 5632, 24 layers | 8192 | `other` (tongyi-qianwen) | configuration |
| Phi-3.5-MoE-instruct | — | 16 experts, 2 per token | `router_aux_loss_coef` 0.0, `router_jitter_noise` 0.01, 32 layers | 131072 | `mit` | configuration |
| OLMoE-1B-7B-0924 | 7B : 1B | 64 experts, 8 per token | `router_aux_loss_coef` 0.01, 16 layers | 4096 | `apache-2.0` | configuration + paper (arXiv 2409.02060) |
| Mixtral-8x7B-v0.1 | 47B : 13B | 8 experts, 2 per token | `router_aux_loss_coef` 0.02, 32 layers | 32768 | `apache-2.0` | configuration + paper (arXiv 2401.04088) |
| Mixtral-8x22B-v0.1 | — | 8 experts, 2 per token | `router_aux_loss_coef` 0.001, 56 layers | 65536 | `apache-2.0` | configuration |
| Llama-4-Scout-17B-16E-Instruct | 109B : 17B | 16 experts | model card: "17 billion parameter model with 16 experts" | 10M (model card) | `llama4` | model card (config.json 401 gated) |
| Qwen3-30B-A3B | — | 128 experts, 8 per token | `moe_intermediate_size` 768, `router_aux_loss_coef` 0.001, `norm_topk_prob` true, 48 layers | 40960 | `apache-2.0` | configuration |
| Qwen3-235B-A22B | — | 128 experts, 8 per token | `moe_intermediate_size` 1536, `router_aux_loss_coef` 0.001, `norm_topk_prob` true, 94 layers | 40960 | `apache-2.0` | configuration |
| gpt-oss-120b | — | 128 experts, 4 per token | `router_aux_loss_coef` 0.9, 36 layers | 131072 | `apache-2.0` | configuration |
| DeepSeek-V3 | 671B : 37B | 256 routed experts + 1 shared, 8 per token | `moe_intermediate_size` 2048, `norm_topk_prob` true, `routed_scaling_factor` 2.5, 61 layers; **auxiliary-loss-free** (no `router_aux_loss_coef` key) | 163840 | code MIT; model "Model Agreement" | configuration + paper (arXiv 2412.19437) |

### 11.1 Reading the table

- **The shared-expert column is not empty in the shipped set.** Qwen1.5-MoE-A2.7B carries a shared-expert width, and DeepSeek-V3 carries a shared expert explicitly (`n_shared_experts` 1). A model with a shared expert has an expert in *every* token's path — the nearest thing to a dense tunable module in a sparse model.
- **The balancing mechanism differs between rows.** Mixtral and Qwen carry an auxiliary-loss coefficient; Phi-3.5-MoE sets it to 0.0 and uses jitter noise; DeepSeek-V3 has no such key and uses an auxiliary-loss-free bias (arXiv 2412.19437). A recipe that assumes one mechanism will not generalise.
- **The parameter figures are versioned.** "47B/13B" belongs to Mixtral 8x7B-v0.1; "671B/37B" belongs to DeepSeek-V3 as reported on 27 Dec 2024. Neither is a property of "MoE". Any figure quoted without its checkpoint is a figure that will go stale.
- **Configurations are the fine-tuner's ground truth.** Where a model card and a paper disagree — Mixtral's 47B (abstract) versus 46.7B (body) — the published `config.json`'s routing fields are what a fine-tuning recipe actually reads, and the discrepancy is recorded rather than resolved.

### 11.2 One model worth naming for a reproducibility-minded reader

OLMoE is the outlier in the set: it is "fully open" — the authors "open-source all aspects of our work: model weights, training data, code, and logs" — and its routing is analysed in the paper, which reports "analyze routing in our model showing high specialization" (arXiv 2409.02060, 3 Sep 2024; v2 3 Mar 2025). **(verified)** For a reader who wants to reproduce a routing histogram or a routing-guided fine-tune on a model whose training is inspectable end to end, OLMoE is the natural first target. That is a statement about *openness*, not about quality.

---
## 12. The Regulated and On-Premises Angle

### 12.1 Data residency

For a regulated institution the interesting question about a sparse model is not the architecture but where the data goes. A fine-tune runs entirely on-premises if the base weights and the training data both live there — this is the air-gapped pattern owned by `technology/on_prem_llm_deployment_guide.md` §18 (air-gapped deployment) and §10 (model licensing for banking). What the MoE adds to the residency picture is only a sizing fact: a sparse model's *total* parameter count determines what must be resident, so a "37B active" model may need the storage, memory and quantisation planning of a far larger dense model (see §8.2). The residency posture is unchanged; the footprint is not.

### 12.2 The auditability of a fine-tuning run

The sibling guides own the audit trail for deployment — the on-prem guide's §23.3 (audit trail) and §16 (model registry and versioning); the instruction-tuning guide's §8 (deployment and iteration). This guide adds one MoE-specific artefact to that trail, and it is the artefact Section 7 argues for: **the routing histogram**. A dense fine-tune's audit trail is the dataset, the hyperparameters, the model hash and the evaluation report. A sparse fine-tune's audit trail should carry one more thing — the per-layer routing profile, pre and post — because that is the record of *what the fine-tune actually did* when the model is sparse. Without it, an auditor asking "which experts does this adapter change, and did that change the routing?" has no answer, and the change-management and versioning machinery of the on-prem guide's §23.4 has nothing to version beyond the adapter file.

| Audit artefact | Dense fine-tune | Sparse fine-tune | Owner |
|---|---|---|---|
| Dataset + provenance + decontamination | Required | Required | instruction-tuning guide §8 |
| Hyperparameters, base model hash | Required | Required | comparison guide §9, §13 |
| Adapter / checkpoint versioning | Required | Required | on-prem guide §16, §23.4 |
| Evaluation report | Required | Required | comparison guide §10, instruction-tuning guide §7 |
| **Routing profile (pre/post)** | *n/a* | **Recommended MoE-specific addition** | **this guide §7, §9** |
| Adapted-expert list + target parameters | *n/a* | **Recommended MoE-specific addition** | **this guide §5, §6** |

### 12.3 Licence terms as published

Licence review is owned by the on-prem guide's §10 ("Open Weights ≠ Open Source"; the licence-review checklist for banking). This guide does not repeat that review; it records only what the repositories publish, as retrieved 2026-10-14, and points the reader to §10 for how to weigh it. The published fields are: Mixtral 8x7B and 8x22B `apache-2.0`; Qwen3-235B-A22B and Qwen3-30B-A3B `apache-2.0`; gpt-oss-120b `apache-2.0`; OLMoE-1B-7B `apache-2.0`; Phi-3.5-MoE-instruct `mit`; Qwen1.5-MoE-A2.7B `other` with `license_name: tongyi-qianwen`; DeepSeek-V3 code under MIT with the model under a "Model Agreement"; Llama-4-Scout under `llama4`. **(verified)** These are the *published* terms; whether they satisfy a given bank's procurement is a question for the guide that owns it, not this one. One MoE-specific note does belong here: an upcycled or fine-tuned model inherits the base's licence terms, and the *experts* are the part of the model most likely to have been copied from the base — a point the software-composition review should not miss when it treats a sparse model as "the base plus an adapter".

### 12.4 What this section deliberately does not do

It does not restate data-sovereignty, compliance-mapping, security or vendor-governance material. Those are the on-prem guide's §1 and §23, in full, and they apply to a sparse model exactly as they apply to a dense one. The regulated reader's takeaway from *this* guide is narrow: the MoE changes the footprint of the run, adds a routing artefact to the audit trail, and adds one licence-review question (what did the upcycle copy) — and leaves every other governance question where it already lived.

---
## 13. The Cymbal Bank Worked Example

*Cymbal Bank is a fictional institution used throughout this guide. The scenario is illustrative — the figures are invented to show a decision, not to report a measurement — and the mechanisms it exercises are the ones sourced in Sections 2–10. No real institution, client or colleague appears.*

### 13.1 The task

Cymbal Bank's trade-finance operations team wants an assistant that marks up letters of credit and shipping documents against the bank's internal compliance rules and flags clauses that need review. The task is classification-with-explanation over long, idiosyncratic documents: a handful of common clause types, and a long tail of rare formats and legacy wordings that together account for a third of the cases and nearly all the risk. The team has a curated, decontaminated labelled set — the ordinary data work the instruction-tuning guide's §8 owns — and a GPU estate that can serve a sparse, open-weight model whose *active* parameters are modest.

### 13.2 The dense instinct

The team's first plan was the dense one: pick a model, LoRA the attention projections (`q_proj`, `k_proj`, `v_proj`, `o_proj`), evaluate on a held-out set, ship. This is the default recipe the comparison guide's §4 documents, and on a dense model it is often enough. On the sparse model it was not.

### 13.3 The first run, and the first surprise

The attention-only LoRA improved the formatting of the assistant's answers — tone, structure, the standard preamble — and did nothing for the long-tail clause types. The loss fell; the long-tail accuracy did not move. In a dense programme this is the signal to raise the data or the rank. Here it was the signal to look at the routing.

Cymbal ran the calibration set through the base model with router logging and produced the histogram. The long-tail clause documents were not spread across the model's experts: in several layers they concentrated on a small subset — a handful of experts took most of the long-tail tokens, and the attention-only LoRA had adapted none of them, because the experts were never in the target set at all. The intersection of §4.1 — "the experts this data reached" ∩ "the experts this task needed" — was computed for the first time, and it was not empty, but the recipe had been blind to it.

### 13.4 The changes the sparsity forced

Three things changed, all of them MoE-specific, none of them about LoRA mechanics:

1. **The target set changed.** The experts most-routed for the long-tail documents were added to the target set via PEFT's `target_parameters`, with a rank scaled down by the expert count so the parameter budget stayed comparable to the dense baseline (PEFT `lora.md`, retrieved 2026-10-14). **(verified)** The un-routed experts stayed out — the MoE-Sieve lesson that adapting the cold experts adds noise for no accuracy.
2. **The router was frozen — deliberately, this time.** Cymbal chose not to train the gate, on the logic of §4.7: the pretrained router already separated the common clauses well, and the risk of moving it was greater than the benefit of it learning the long tail in a few hundred steps. This was a *decision*, recorded, not an accident.
3. **The evaluation changed shape.** The task accuracy number was joined by a routing histogram, taken pre- and post-fine-tune, as the audit artefact of §12.2.

### 13.5 The one decision the MoE-ness changed

**The decision of whether the fine-tune was finished.** A dense programme answers this question with the evaluation set: if the held-out accuracy is good, the fine-tune is done. Cymbal's sparse programme could not, and a dense-shaped programme would have got it wrong here — it would have shipped on the eval pass. The routing histogram showed that the second fine-tune, which hit its accuracy target on the long tail, had also *concentrated* the long-tail routing onto a still-smaller set of experts than before. The answers were right; the *serving* was now uneven, and the all-to-all of §8.3 had a hot expert that the latency budget had never been sized for. The program was not finished at the eval pass; it was finished only after the routing was checked, the concentration understood, and the capacity plan re-measured for the observed hot expert rather than for a uniform spread.

That is the whole distinction in one decision. Accuracy is necessary for a sparse model and insufficient. The extra question — *where did the tokens go, and did that change?* — is the one the dense programme had no instrument for and the sparse programme could not do without.

### 13.6 The ending, against the thesis

Cymbal's fine-tune succeeded. It succeeded by a route a dense-model programme would not have walked: it found the long-tail experts through a routing histogram, adapted the experts the data actually reached, froze the router on purpose, and refused to call the run finished on accuracy alone. None of that was exotic; all of it was a single extra instrumentation habit — measure the routing, and treat the sparse model as two systems, weights and router, rather than one. The thesis holds in the ordinary case as much as the pathological one: **every fine-tuning run on a sparse model is also a routing run.**

---
## 14. The Anti-Patterns

Each anti-pattern is named, given its tell, and given the correction. They are the recurring mistakes this guide exists to pre-empt.

### 14.1 "The MoE is just a big model — use the dense recipe"

The tell: a fine-tuning plan that lists attention projections and hyperparameters and never mentions experts or routing. The correction: for a sparse model the target set is a *choice* (attention, experts, router), and the default attention-only recipe is the failure mode of §10.3. Decide the target set explicitly, and write down whether the router is trained or frozen.

### 14.2 "Adapt every expert uniformly"

The tell: `target_parameters` set to the fused expert tensors with one rank for all experts, no routing profile taken. The correction: MoE-Sieve shows the top-25% most-routed experts per layer match full LoRA within ±1pp while cutting trainable parameters 70-73%, and that cold-expert adaptation adds "gradient noise without improving accuracy" (arXiv 2603.24044). **(verified)** Uniform adaptation is not wrong; it is unexamined.

### 14.3 "It passed evaluation, ship it"

The tell: a sparse fine-tune with an accuracy report and no routing report. The correction: accuracy and routing are different variables (§9.4); a collapsed router passes accuracy and fails throughput. Add the pre/post routing histogram to the completion criteria, not to the debugging toolkit.

### 14.4 "Let the router learn the domain — more capacity must help"

The tell: the gate added to the tunable set with no plan for load balance. The correction: a trained router can collapse (§3, §10.1). If the router is trained, the balance mechanism the checkpoint shipped — auxiliary loss, jitter noise, or adaptive bias — is live, and the routing histogram is the instrument that says whether it held.

### 14.5 "Leave the adapters unmerged — merging can wait"

The tell: an un-merged LoRA on the experts in a serving path. The correction: PEFT "always materializes the LoRA contribution for _each expert_ even if only a small amount of experts is required", so an un-merged expert adapter "can result in a substantial slowdown at inference time" (PEFT `lora.md`, retrieved 2026-10-14). **(verified)** On a sparse model, merging is what preserves the sparsity; leaving adapters loose defeats it.

### 14.6 "Upcycle is a free lunch — fine-tune the upcycled model straight away"

The tell: an upcycled checkpoint fed directly into a domain fine-tune. The correction: upcycling initialises the experts from a dense checkpoint (arXiv 2212.05055) and needs its post-upcycle training to make the experts distinguishable before the router can be informational (§10.6). Two stages, not one.

### 14.7 "A benchmark number proves the sparse model is good"

The tell: a quality ranking built from benchmark scores, or a parameter count treated as a quality claim. The correction: this guide prints no benchmark number for a reason — it did not retrieve one from a primary source, and parameter counts are not quality (§11). Configuration facts are for planning a fine-tune, not for choosing a winner.

### 14.8 "The router is infrastructure, not a training concern"

The tell: a fine-tuning runbook in which the router appears nowhere — not in targets, not in checks, not in the audit trail. The correction: the router is the module that decides which parameters ever see the data (§4.2–§4.4); it belongs in the target-set decision, the routing check and the audit artefact. Omitting it is how a sparse fine-tune fails while looking successful.

### 14.9 "One dataset, one adapter, done"

The tell: a single adapter trained to serve two domains that route differently, with no Jaccard check. The correction: near-disjoint routing did not prevent negative transfer between a code and a biomedical domain, because the interference lived in the shared adapter subspace (arXiv 2609.03150). **(verified)** When domains contend, separate the adapters or separate the experts deliberately.

---
## 15. The Claims Audit

Every architecture fact and every load-bearing claim in this guide, with its source, the date of the source, and the *kind* of source (paper, model card, configuration, framework documentation, vendor claim). "Paper" here means the arXiv abstract page and, where marked, the paper's full text; "configuration" means a published `config.json`; all retrievals were made on 2026-10-14. The final row is the hazard-control record.

### 15.1 Architecture and mechanism claims

| Claim as stated | Source (URL) | Date | Kind |
|---|---|---|---|
| Origin of the sparsely-gated MoE layer; up to thousands of FFN experts; trainable gating network; up to 137B params between stacked LSTMs | arXiv 1701.06538 | submitted 23 Jan 2017 | paper (abstract) |
| Auxiliary loss $\ell_{aux}$ added to the loss with constant multiplier $k$; capacity overflow / under-utilisation language | arXiv 2006.16668 | 30 Jun 2020 | paper (full text) |
| GShard scaled sparse MoE beyond 600B params; 2048 TPU v3, 4 days | arXiv 2006.16668 | 30 Jun 2020 | paper (abstract) |
| Top-1 routing ("highest router probability"); capacity factor = (total_tokens / num_experts) × capacity_factor; overflow tokens dropped, typically <1%; auxiliary differentiable load-balancing loss; bfloat16 sparse training; up to trillion-param models | arXiv 2101.03961 | 11 Jan 2021 (v3 16 Jun 2022) | paper (full text) |
| Mixtral: 8 FFN experts per layer; router selects 2 per token per layer; 47B params / 13B active; 32k context (body gives 46.7B — discrepancy noted) | arXiv 2401.04088 | 8 Jan 2024 | paper (abstract) |
| DeepSeekMoE: fine-grained segmentation (mN experts, activate mK); shared-expert isolation ($K_s$) | arXiv 2401.06066 | 11 Jan 2024 | paper (abstract) |
| DeepSeek-V3: 671B total / 37B activated per token; MLA + DeepSeekMoE; auxiliary-loss-free load balancing; multi-token prediction; 14.8T tokens; 2.788M H800 GPU-hours | arXiv 2412.19437 | 27 Dec 2024 (v2 18 Feb 2025) | paper (abstract) |
| Expert-choice routing: experts select top-k tokens; variable experts per token; fixed bucket size per expert; >2× faster convergence | arXiv 2202.09368 | 18 Feb 2022 (v2 14 Oct 2022) | paper (abstract) |
| Sparse upcycling: initialise sparse MoE from a dense checkpoint; beats dense on ~50% of the dense pretraining budget | arXiv 2212.05055 | 9 Dec 2022 (v2 17 Feb 2023) | paper (abstract) |
| DeepSpeed-MoE: up to 3.7× size reduction; 5× training-cost saving for auto-regressive LMs | arXiv 2201.05596 | 14 Jan 2022 (v2 21 Jul 2022) | paper (abstract) |
| ST-MoE: training instabilities and "uncertain quality during fine-tuning"; 269B params at ~32B-dense compute | arXiv 2202.08906 | 17 Feb 2022 (v2 29 Apr 2022) | paper (abstract) |
| OLMoE-1B-7B: 7B params / 1B per token; 5T tokens; routing analysis showing high specialisation; fully open (weights, data, code, logs) | arXiv 2409.02060 | 3 Sep 2024 (v2 3 Mar 2025) | paper (abstract) |

### 15.2 Fine-tuning-on-MoE claims

| Claim as stated | Source (URL) | Date | Kind |
|---|---|---|---|
| PERFT: unified framework integrating PEFT into the MoE mechanism; design dimensions; adapts OLMoE-1B-7B and Mixtral-8x7B; "intriguing dynamics" | arXiv 2411.08212 | 12 Nov 2024 | paper (abstract) |
| MoE-Sieve: routing highly skewed ("cold" experts); adapt top-25% routed experts per layer; within ±1pp of full LoRA; 70-73% fewer trainable params; up to 50% less wall-clock; random selection ~2.5pp worse; non-monotonic expert-count/variance; greedy budget ≤ uniform top-k | arXiv 2603.24044 | 25 Mar 2026 | paper (abstract) |
| "Routing Is Not Enough": near-disjoint routing did not prevent negative transfer; introduces Jaccard routing overlap and adapter-gradient cosine similarity; contention in the shared low-rank subspace; SpawnLoRA keeps the router fixed | arXiv 2609.03150 | 2 Sep 2026 | paper (abstract) |
| DR-LoRA: heterogeneous per-expert rank from routing frequency + gradient-based rank importance; uniform rank under-provisions task-relevant experts | arXiv 2601.04823 | 8 Jan 2026 (v6 31 Aug 2026) | preprint (abstract) — v6 postdates v1 |
| Mixture of LoRA Experts (MoLE): hierarchical mixture over LoRA layers | arXiv 2404.13628 | 21 Apr 2024 | paper (abstract) |
| MoELoRA: LoRA as mixture of experts; contrastive learning to mitigate random routing | arXiv 2402.12851 | 20 Feb 2024 | paper (abstract) |

### 15.3 Configuration and licence claims (all retrieved 2026-10-14)

| Claim as stated | Source | Date | Kind |
|---|---|---|---|
| Mixtral-8x7B-v0.1: 8 experts, 2 per token, `router_aux_loss_coef` 0.02, 32 layers, ctx 32768 | `mistralai/Mixtral-8x7B-v0.1` config.json | retrieved 2026-10-14 | configuration |
| Mixtral-8x22B-v0.1: 8 experts, 2 per token, `router_aux_loss_coef` 0.001, 56 layers, ctx 65536 | config.json | retrieved 2026-10-14 | configuration |
| Qwen3-235B-A22B: 128 experts, 8 per token, `norm_topk_prob` true, `router_aux_loss_coef` 0.001, `moe_intermediate_size` 1536, 94 layers, ctx 40960 | config.json | retrieved 2026-10-14 | configuration |
| Qwen3-30B-A3B: 128 experts, 8 per token, `moe_intermediate_size` 768, 48 layers, ctx 40960 | config.json | retrieved 2026-10-14 | configuration |
| Qwen1.5-MoE-A2.7B: 60 experts, 4 per token, shared expert `shared_expert_intermediate_size` 5632, `moe_intermediate_size` 1408, 24 layers, ctx 8192 | config.json | retrieved 2026-10-14 | configuration |
| gpt-oss-120b: 128 experts, 4 per token, `router_aux_loss_coef` 0.9, 36 layers, ctx 131072 | config.json | retrieved 2026-10-14 | configuration |
| OLMoE-1B-7B-0924: 64 experts, 8 per token, `router_aux_loss_coef` 0.01, 16 layers, ctx 4096 | config.json | retrieved 2026-10-14 | configuration |
| DeepSeek-V3: 256 routed experts, 1 shared, 8 per token, `routed_scaling_factor` 2.5, no `router_aux_loss_coef` key, ctx 163840 | config.json | retrieved 2026-10-14 | configuration |
| Phi-3.5-MoE-instruct: 16 experts, 2 per token, `router_aux_loss_coef` 0.0, `router_jitter_noise` 0.01, ctx 131072 | config.json | retrieved 2026-10-14 | configuration |
| Llama-4-Scout-17B-16E-Instruct: 17B activated / 109B total, 16 experts, context length 10M, licence `llama4` (Llama 4 Community License, Version Effective Date 5 Apr 2025; HF repo created 2 Apr 2025) | HF model card (config.json 401, gated) + HF model API metadata | retrieved 2026-10-14 | model card |
| Licences as published: Mixtral ×2 `apache-2.0`; Qwen3-235B/30B `apache-2.0`; gpt-oss-120b `apache-2.0`; OLMoE `apache-2.0`; Phi-3.5-MoE `mit`; Qwen1.5-MoE-A2.7B `other` (`tongyi-qianwen`); DeepSeek-V3 code MIT / model "Model Agreement"; Llama-4-Scout `llama4` | HF model-card metadata | retrieved 2026-10-14 | model card / repository |

### 15.4 Framework-documentation claims (all retrieved 2026-10-14)

| Claim as stated | Source | Kind |
|---|---|---|
| PEFT `target_parameters` for MoE expert `nn.Parameter` tensors; 3-D param's 0th dim is the expert dim; PEFT materialises the LoRA contribution for *each* expert → inference slowdown; recommend `merge_adapter()`/`merge_and_unload()`; `rank_pattern` with `effective_r = max(1, r // num_experts)`; multiple `target_parameters` adapters must target the same params; vLLM supports fused MoE expert layers since v0.11.2; KappaTuneSelector scans fused 3-D MoE params | PEFT `package_reference/lora.md` | framework documentation |
| vLLM EP: experts on separate GPUs; `EP_SIZE = TP_SIZE × DP_SIZE`; `--enable-expert-parallel`; expert layers sharded across EP ranks; `--all2all-backend` options (allgather_reducescatter / deepep_high_throughput / deepep_low_latency / flashinfer_nvlink_*) | vLLM *Expert Parallel Deployment* | framework documentation |
| EP distributes experts across GPUs, applied only to expert layers; `expert_model_parallel_size`; Expert Tensor Parallelism `expert_tensor_parallel_size`; DeepEP / HybridEP dispatchers | Megatron Bridge *Parallelisms Guide* | framework documentation |
| `--moe-train-capacity-factor` / `--moe-eval-capacity-factor` / `--moe-min-capacity` set how many tokens a single expert handles; `--moe-loss-coeff` (0.01 a good setting); larger capacity → better convergence but slower training | DeepSpeed-MoE tutorial | framework documentation / vendor claim |

### 15.5 Hazard control

| Control | Record |
|---|---|
| **No architecture figure is printed without a primary source** | **Confirmed.** Every expert count, active/total parameter figure, top-k, auxiliary-loss coefficient, context window and layer count in this guide appears above with an arXiv paper, a published `config.json`, or a published model card as its source. No figure was written from memory or from a secondary summary. |
| Version-dependence of architecture facts | Stated in the front matter and in §11.1; every figure carries its checkpoint and date. |
| Benchmark scores | **None printed.** No benchmark score was retrieved from a primary source, so none appears in the body. |
| Throughput numbers | **None printed as measurement.** The parallel-framework facts (EP_SIZE, capacity knobs) are documentation, not measured throughput. |
| Discrepancies recorded, not resolved | Mixtral 47B (abstract) vs 46.7B (body); the guide cites the abstract and records the body value. |

---
## 16. What Could Not Be Verified, the Glossary, the Cross-References and the Closing Summary

### 16.1 What could not be verified

These are explicit negatives. Where a figure could not be retrieved from a primary source, no figure is printed in the body, and the negative is recorded here.

- **Llama-4-Scout-17B-16E-Instruct `config.json` — not retrievable (HTTP 401, gated).** The routing configuration (top-k, routers, layer counts) for Llama-4 could **not** be read from a published config. The figures used in §11 (17B activated / 109B total, 16 experts, 10M context, licence `llama4`) come from the *public model card*, retrieved 2026-10-14, and are labelled as model-card figures, not configuration. **(unverified-flag)** — no Llama-4 routing figure from a `config.json` appears anywhere in this guide.
- **The PEFT "MoE models" developer-guide page — not found (negative finding).** The documentation path named as available (`developer_guides/moe_models`, v0.21.0) resolves to a redirect notice, and the current `huggingface/peft` repository tree contains no `moe_models.md`. The MoE guidance that PEFT actually publishes lives in `docs/source/package_reference/lora.md` (the `target_parameters` section), which is what this guide cites. **(negative finding)**
- **vLLM `/features/expert_parallelism.html` — 404 (negative finding).** The expert-parallelism documentation that loads is `/en/latest/serving/expert_parallel_deployment/`, which this guide cites. The path in the hint did not resolve. **(negative finding)**
- **`web_search` — unreliable on this host.** Web search returned empty results. All sources were obtained by direct URL extraction and `web_extract`. The absence of search-derived material is a **tool limitation**, not evidence that no such material exists. **(unverified-flag)**
- **Benchmark scores — none retrieved, none printed.** No benchmark score for any model in §11 was obtained from a primary source, so the guide prints no benchmark number and offers no quality ranking. **(negative finding)**
- **Measured throughput — none printed.** The parallelism figures (EP sizing, capacity knobs) are framework documentation, not measured throughput; the guide distinguishes them and claims no throughput number. **(negative finding)**
- **Router z-loss as a named mechanism — not cited to a primary source.** The concept appears in the sparse-model literature, but no primary-source definition was retrieved in this pass; the guide does not use the term. **(negative finding)**
- **Unsloth / Axolotl / LlamaFactory MoE fine-tuning documentation — not retrieved in this pass.** Whether these tools document MoE-specific targeting was not established, so the guide makes no claim about them. **(unverified-flag)**
- **Mixtral parameter discrepancy — recorded, not resolved.** The abstract gives 47B; the body of the same paper gives 46.7B (arXiv 2401.04088, 8 Jan 2024). The guide cites the abstract and records the body value rather than choosing. **(flagged)**

### 16.2 Glossary

- **Active parameters.** The parameters used to process one token — the k experts selected plus the shared and non-expert weights. DeepSeek-V3: 37B active (arXiv 2412.19437).
- **Auxiliary-loss-free load balancing.** Maintaining balance with an adaptive bias on the router rather than an added loss term (DeepSeek-V3, arXiv 2412.19437).
- **Auxiliary load-balancing loss.** A differentiable term added to the training loss to push the router toward even expert use (GShard, arXiv 2006.16668; Switch Transformers, arXiv 2101.03961). Its coefficient is a config value: e.g. 0.02 (Mixtral-8x7B), 0.9 (gpt-oss-120b).
- **Capacity factor.** The per-expert token ceiling: (total_tokens / num_experts) × capacity_factor; overflow tokens are dropped (Switch Transformers, arXiv 2101.03961).
- **Expert.** One feed-forward sub-network in an MoE layer that competes with its siblings to process a token.
- **Expert-choice routing.** Routing in which experts select the top-k tokens, giving each expert a fixed bucket and each token a variable number of experts (arXiv 2202.09368).
- **Expert parallelism (EP).** Distributing a model's experts across devices; applies only to expert layers; `EP_SIZE = TP_SIZE × DP_SIZE` in vLLM.
- **Fine-grained expert.** A smaller expert produced by segmenting a conventional expert, so more can be activated per token (DeepSeekMoE, arXiv 2401.06066).
- **Jaccard routing overlap.** A measure of expert sharing between two domains' routing (arXiv 2609.03150).
- **Load balancing.** Even distribution of tokens across experts; its failure is collapse (concentration on few experts) or starvation (under-trained experts).
- **MoE (mixture of experts).** An architecture replacing the dense feed-forward block with a set of experts plus a router. The acronym-as-Ministry collision is noted in §1.2; in this guide it means mixture of experts only.
- **Routing histogram.** A per-layer count of the tokens each expert received, taken by running a calibration set through the model with router logging (this guide, §7.2).
- **Router / gate.** The learned module that scores tokens against experts and selects which run.
- **Shared expert.** An expert that runs for every token, carrying common knowledge (DeepSeekMoE, arXiv 2401.06066).
- **Sparse activation.** Only the selected experts compute, decoupling parameter count from per-token cost.
- **`target_parameters`.** PEFT's mechanism for applying LoRA to `nn.Parameter` tensors (e.g. fused expert weights) rather than `nn.Linear` modules (PEFT `lora.md`).
- **Token dropping.** Discarding tokens that exceed an expert's capacity (Switch Transformers, arXiv 2101.03961).
- **Total parameters.** All parameters of the model, resident regardless of which are active. DeepSeek-V3: 671B total (arXiv 2412.19437).
- **Upcycling.** Initialising a sparse model from a dense checkpoint (arXiv 2212.05055).

### 16.3 Cross-references

**The four owner guides** (crossed, never re-derived):

- `technology/ai_llm/fine_tuning_frameworks_comparison_guide.md` — fine-tuning definition; full vs PEFT (§3); PEFT methods (§4); framework comparison (§5); RLHF/alignment (§6); quantisation-aware fine-tuning (§7); data preparation (§8); infrastructure (§9); evaluation (§10); fine-tune vs RAG (§11); framework selection (§13).
- `technology/ai_llm/llm_instruction_tuning_guide.md` — SFT mechanics, templates, masking (§4); SFT vs RLHF vs DPO (§5); PEFT for instruction tuning (§6); IFEval (§7); curation, decontamination, iteration (§8); a banking-assistant example (§9); glossary (§11).
- `technology/ai_llm/llm_compression_alternatives_guide.md` — quantisation frameworks (§3); pruning and sparsity (§4); distillation (§5); model merging (§6); formats (§9); benchmarks (§11); emerging trends incl. MoE Compression (§12.2) and QLoRA (§12.6).
- `technology/on_prem_llm_deployment_guide.md` — MoE vs Dense Architectures (§9); model licensing for banking (§10); quantisation for on-prem (§11); Fine-Tuning for On-Prem (§12); serving stack (§13); serving patterns (§14); air-gapped deployment (§18); governance and audit trail (§23); pitfalls (§25).

**Deployment and operations:** `technology/on_prem_llm_deployment_guide.md` (the serving stack this guide's §8 cross-references rather than restates).

**Banking and regulated-AI context** (for the reader applying this in an institution, as the Cymbal example does):

- `technology/ai_llm/multi_agent_banking_guide.md` — agent architectures for banking.
- `technology/ai_llm/llm_agent_use_cases_finance_guide.md` — finance use cases for LLM agents.
- `banking/ai_genai_banking_compliance_guide.md` — compliance framing for generative AI in banking.
- `banking/ai_risk_register_guide.md` — the risk register that should carry a sparse model's routing artefact.
- `banking/trade_finance_guide.md` — the trade-finance domain the Cymbal example draws on.

### 16.4 Closing summary

This guide has made one argument in sixteen sections. A mixture-of-experts model is a model whose computation is chosen per token by a learned router; consequently, fine-tuning one is two coupled processes — the adaptation of weights and the routing that decides which weights ever see the data — and conventional fine-tuning practice instruments only the first. Sections 2 and 3 built the mechanism from primary sources; Section 4 developed why fine-tuning a sparse model is not fine-tuning a dense one, and stated the honest counter-case where the distinction is theoretical; Sections 5 and 6 decided which modules and which experts to adapt; Section 7 made the routing histogram a first-class dataset artefact; Sections 8 and 9 put infrastructure and evaluation around it; Sections 10 and 14 catalogued how it goes wrong. Sections 11 and 12 gave the open models and the regulated framing, factually and without ranking, and Section 13 showed one decision — whether a fine-tune is finished — that a sparse programme gets right and a dense-shaped one gets wrong. Every architecture figure was sourced and dated or omitted. The claim that survives all of it is the one that opened the guide and closes it now:

every fine-tuning run on a sparse model is also a routing run.
