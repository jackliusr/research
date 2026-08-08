# Chinese Text Tokenization (中文分词): Comprehensive Guide

> **Author:** Jack Liu Shurui · **Role:** Solution Architect, Crédit Agricole CIB
> **Repo:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Series:** NLP/LLM Engineering Guides
> **Companion Guides:** [RAG Frameworks Comparison](rag_frameworks_comparison_guide.md) · [Advanced RAG Techniques](advanced_rag_techniques_guide.md) · [Beyond RAG — Agent Memory](beyond_rag_guide.md) · [Vector Databases](vector_databases_guide.md) · [RAG vs Long-Context LLMs](rag_vs_long_context_llms_guide.md) · [LLM Optimization Complete Guide](llm_optimization_complete_guide.md) · [On-Prem LLM Deployment](../on_prem_llm_deployment_guide.md) · [Chinese Bank Core Systems](../../banking/chinese_bank_core_systems_guide.md) · [Tokenized Assets (banking)](../../banking/tokenized_assets_guide.md)
> **Last Updated:** August 2026

> **Disambiguation note.** This guide is about **Chinese *text* tokenization (中文分词)** — the NLP/LLM task of splitting Chinese text into words, characters, and model tokens. It is **not** about financial *asset* tokenization (tokenized bonds, funds, deposits) — that topic lives in the banking series, [Tokenized Assets Guide](../../banking/tokenized_assets_guide.md). The two topics share a word ("token") but nothing else. Where this guide says "tokenizer," it means the component that turns text into model vocabulary IDs; where the banking guide says "tokenization," it means representing a financial asset on a ledger.

---

## Table of Contents

1. [Introduction: Two Different Kinds of "Tokenization"](#1-introduction-two-different-kinds-of-tokenization)
2. [The Chinese Segmentation Challenge](#2-the-chinese-segmentation-challenge)
3. [Classical Approaches I: Dictionary-Based Segmentation](#3-classical-approaches-i-dictionary-based-segmentation)
4. [Classical Approaches II: Statistical Segmentation (HMM, N-gram, CRF)](#4-classical-approaches-ii-statistical-segmentation-hmm-n-gram-crf)
5. [Evaluation: Metrics, Corpora, and the SIGHAN Bakeoffs](#5-evaluation-metrics-corpora-and-the-sighan-bakeoffs)
6. [Neural Approaches: From BiLSTM-CRF to BERT](#6-neural-approaches-from-bilstm-crf-to-bert)
7. [The LLM Era: Subword Tokenizers and SentencePiece](#7-the-llm-era-subword-tokenizers-and-sentencepiece)
8. [Chinese LLM Tokenizers: Qwen, DeepSeek, Baichuan, GLM and Friends](#8-chinese-llm-tokenizers-qwen-deepseek-baichuan-glm-and-friends)
9. [Token-Efficiency Economics: Is Chinese Token-Hungry?](#9-token-efficiency-economics-is-chinese-token-hungry)
10. [Libraries and Tools](#10-libraries-and-tools)
11. [Chinese Tokenization in LLM Applications (RAG, Cost, Fine-Tuning)](#11-chinese-tokenization-in-llm-applications-rag-cost-fine-tuning)
12. [Practical Guidance](#12-practical-guidance)
13. [Worked Example: One Sentence, Five Tokenizers](#13-worked-example-one-sentence-five-tokenizers)
14. [The Future (2026 and Beyond)](#14-the-future-2026-and-beyond)
15. [Glossary](#15-glossary)
16. [Verification Notes & Sources](#16-verification-notes--sources)

---

## 1. Introduction: Two Different Kinds of "Tokenization"

**Chinese is the only major world language whose *first* NLP step is itself a hard NLP problem.** English, French, German — their words are already delimited by whitespace; tokenization is mostly clerical. Chinese has **no word boundaries at all**: the text is a continuous stream of ideographs (汉字), and *deciding where one word ends and the next begins* is a research problem that consumed the Chinese NLP community for three decades — from hand-built dictionaries in the 1980s, through statistical models (HMM, CRF) in the 2000s, to neural sequence labeling (BiLSTM-CRF, BERT) in the 2010s. That task is called **Chinese word segmentation (CWS, 中文分词)**, and this guide is about it.

Then the LLM era arrived and quietly *relabeled* the problem. A "tokenizer" in the LLM world is not a word segmenter; it is a **subword vocabulary encoder** — Byte Pair Encoding (BPE), WordPiece, or Unigram — that maps any Unicode text to integer IDs, usually with **no linguistic notion of "word" at all**. Modern LLM tokenizers do not segment Chinese into 词 (words); they merge characters into whatever byte/character n-grams minimize average sequence length on the training corpus.

So there are **two distinct layers** to keep apart throughout this guide:

| Layer | Task | Unit | Question it answers | Example tools |
|---|---|---|---|---|
| **NLP word segmentation (CWS)** | 分词 — segment text into words | 词 (word) | "What are the words?" | jieba, HanLP, LTP, THULAC, pkuseg |
| **LLM subword tokenization** | 分片 — encode text into vocab IDs | token (subword/byte) | "What are the model's input IDs?" | SentencePiece, tiktoken, HF tokenizers |

The two layers interact: a Chinese-optimized LLM vocabulary (Qwen, DeepSeek) looks a lot like a statistical segmenter's word list — it contains tens of thousands of common Chinese words and bigrams as single tokens — while an English-centric vocabulary (GPT-2, Llama-2) treats most Chinese characters as raw UTF-8 bytes and burns 2–3 tokens per character. **That difference is the entire economics story of this guide** (§9).

For a bank's AI platform team, this matters concretely: Chinese text is the input to search indexes, RAG pipelines, prompt budgets, and fine-tuning runs; whether that text is segmented into words (jieba) or packed into subword tokens (Qwen tokenizer) changes retrieval quality, token cost, context-window utilization, and model choice. The worked example in §13 runs the same Chinese sentence through jieba, Qwen2.5, Llama-3, and two OpenAI tokenizers with real measured counts.

---

## 2. The Chinese Segmentation Challenge

### 2.1 No word boundaries: segmentation is the first step

English text arrives pre-segmented: `The customer opened an account.` The spaces are word boundaries, and punctuation does the rest. **Chinese text arrives as one unbroken glyph stream**: `客户在银行开立账户`. There is no space, no capitalization, no morphology, no inflection to lean on. A Chinese reader segments effortlessly because they know the language; a machine must *infer* the segmentation, and the inference is genuinely ambiguous (§2.3). This is why CWS is not a preprocessing nicety but the **foundational first step** of the classical Chinese NLP pipeline — part-of-speech tagging, named-entity recognition, parsing, and information retrieval were all historically built *on top of* a segmentation layer, so every downstream error inherits segmentation errors.

### 2.2 字 (characters) vs 词 (words): two units, two philosophies

- **字 (zì)** — the character. Each Chinese character is one syllable, roughly one morpheme, written as one glyph. There are ~3,500 common characters (GB2312) / ~20,000 in common use (GBK), ~50,000+ in the full Unicode CJK block (with extensions, >90,000).
- **词 (cí)** — the word. A word is one or more characters forming a lexical unit: 银行 (bank, 2 chars), 跨境支付 (cross-border payment, 4 chars). Modern Chinese is overwhelmingly **disyllabic** — the majority of words are two characters, which is why "2-character bigram" is the natural unit for Chinese vocabularies.

The character vs. word distinction drives a **design fork** that runs through the entire history of the field:

- **Character-based processing** treats each 字 as an atomic unit. It sidesteps segmentation entirely, at the cost of losing word-level information (and blowing up sequence length — every word becomes 2 characters).
- **Word-based processing** needs a segmenter first, but then gives the downstream model the natural unit (and, historically, much shorter sequences).

Modern neural CWS found a synthesis: **treat segmentation as character-level sequence labeling** — each character gets a tag saying "begin/middle/end/single of a word" (the BEMS scheme, §4.1) — so the model consumes characters but emits words. LLM tokenizers went further and dropped the word concept entirely (§7).

### 2.3 歧义 (ambiguity): the segmentation trap

The reason segmentation is a *problem* and not a lookup is ambiguity. Two classic types:

1. **Overlap ambiguity (交集型歧义)**: a substring could belong to the word before it or the word after it. The most famous example in the field:

   **`南京市长江大桥`** (Nanjing Yangtze River Bridge)

   - Correct reading (the bridge): `南京市 / 长江大桥` — "Nanjing / Yangtze River Bridge"
   - Alternative reading (a person): `南京 / 市长 / 江大桥` — "Nanjing / Mayor / Jiang Daqiao" — 江大桥 is a plausible personal name!
   - Third reading: `南京市 / 长江 / 大桥`

   A segmenter must pick one. (jieba, running today, picks `南京市 / 长江大桥` — see §13.1.) Other famous overlap cases: `研究生命起源` → `研究 / 生命 / 起源` ("research the origin of life") vs `研究生 / 命 / 起源` ("graduate student / fate / origin" — nonsensical but parseable); `乒乓球拍卖完了` → `乒乓球 / 拍卖 / 完了` ("the ping-pong balls were auctioned off") vs `乒乓球拍 / 卖完了` ("the ping-pong paddles are sold out") — both readings are *grammatical and plausible*, which is the worst kind of ambiguity.

2. **Combination ambiguity (组合型歧义)**: a character sequence is a word on its own *and* can be split into smaller words. `他说的确实在理` — `确实` is a word ("indeed"), but `确 / 实` is also a valid split in other contexts (`确定事实` — "establish facts"). Combination ambiguity is context-dependent: the same string must be segmented differently in different sentences.

Ambiguity is why dictionary lookup alone fails: a perfect dictionary still cannot tell you which of several valid segmentations the *context* demands. That decision requires statistics or learned models (§4).

### 2.4 OOV: unknown words (未登录词)

Every segmenter meets words it has never seen: person names (人名), place names (地名), organizations (机构名), new internet vocabulary (网络新词: 内卷, 破防, 躺平), foreign transliterations (音译词: 麦当劳, 特朗普), and domain jargon (banking: 跨境人民币, 同业拆借). The out-of-vocabulary (OOV) rate on real news text is often 5–10% of *word tokens* — and because unknown words are often exactly the informative ones (names, new terms), **OOV recall is the single most important quality axis** for real-world segmentation (§5.3). OOV handling is what separates the classical approaches: pure dictionary matching has *zero* OOV ability; HMM/CRF models can infer unseen words from character patterns (a 姓 + 名 pattern like `小明` is learnable); neural models do this best. jieba's HMM, for example, exists precisely to rescue OOV words the dictionary misses (§10.1): with HMM on, `小明` is one word; with HMM off, it degrades to `小 / 明`.

### 2.5 Granularity: coarse vs fine (粗细粒度)

Segmentation is not one task with one right answer — it is a **family of tasks parameterized by granularity**. The same text can legitimately be segmented coarsely (`清华大学` as one word) or finely (`清华 / 大学`). Standards differ by corpus and by downstream need:

- **Coarse (大粒度)**: longer units; better for topic/keyword extraction; matches how Chinese dictionaries define 词.
- **Fine (细粒度)**: shorter units; better for search recall and for feeding character-adjacent models; jieba's `lcut_for_search` mode emits both coarse and fine units so a search engine can index all of them.
- **Sub-word / character granularity**: every 字 is a token; the natural choice for LLM vocabularies (§7.3).

The granularity choice is a *product decision*, not a correctness decision — which is why CWS benchmark scores (which measure agreement with one gold standard at one granularity) tell only part of the story. In the LLM era the question mostly evaporates: the tokenizer defines its own granularity by construction, and applications adapt to it.

### 2.6 Chinese vs English tokenization, then and now

| Aspect | English | Chinese |
|---|---|---|
| Word boundaries | Whitespace + punctuation (trivial) | None — must be inferred (hard) |
| Classical "tokenization" | Split on spaces, strip punctuation | Full segmentation problem (CWS) |
| Character inventory | 26 letters + punctuation | 3,500–50,000+ ideographs |
| Morphology | Rich inflection (walk/walked/walking) | No inflection; word = char sequence |
| Word length | Mean ~5 letters; variable | Mean ~2 characters; compact |
| Ambiguity | Rare (hyphenation, "New York") | Pervasive (交集/组合歧义) |
| OOV problem | New words, names | Same, plus the segmentation of unseen words is itself ambiguous |
| LLM-era tokenizer | Subword BPE: word → 1–3 tokens | Char/byte BPE: char → 1–3 tokens (§7.3) |

**The historical summary:** for English, "tokenization" was always a boring preprocessing step; for Chinese, it was *the* research problem. When the LLM era replaced linguistic tokens with subword vocabularies, English tokenization stayed boring — and Chinese tokenization got *replaced*: modern LLMs no longer segment Chinese into words at all, they encode characters (and frequent bigrams) directly (§7–§8). The classical CWS pipeline still matters for search, NER, and lexicon-driven tasks (§11–§12), but the frontier moved to vocabulary design.

---

## 3. Classical Approaches I: Dictionary-Based Segmentation

### 3.1 The lexicon (词典)

The oldest and still most practical idea: keep a **dictionary/lexicon** of words, scan the text, and match. Everything downstream — quality, speed, coverage — hinges on the dictionary. Chinese segmentation dictionaries are large: jieba ships ~350K entries (words + frequencies), HanLP and LTP use dictionaries of comparable scale. Dictionary quality determines the *ceiling* for matching accuracy; the matching *algorithm* determines how close you get.

### 3.2 Maximum matching: FMM, BMM, BM

Given a dictionary, the classic greedy algorithm is **maximum matching**: at each position, try the *longest* dictionary word that starts there.

- **Forward maximum matching (FMM, 正向最大匹配)**: scan left to right; at each position, check dictionary entries up to a maximum word length (typically 5–8 chars), take the longest match, advance.
- **Backward maximum matching (BMM, 逆向最大匹配)**: same idea, right to left. BMM is often more accurate than FMM for Chinese because many ambiguous strings resolve in favor of the *final* characters (e.g., 南京市长江大桥 resolves to 南京市/长江大桥 when scanning backward — the greedy forward scan can strand characters at the end of the string).
- **Bidirectional / best match (BM, 双向最大匹配)**: run both directions, compare results. Heuristics arbitrate: fewer words wins; if tied, fewer single-character words wins; if still tied, prefer BMM. BM is a real quality jump over either direction alone and was the workhorse of early practical segmenters.

**Example** (with `南京市长江大桥`): FMM might produce `南京市 / 长江 / 大桥` or `南京市 / 长江大桥` depending on dictionary entries; BMM robustly yields `南京市 / 长江大桥`. Both are dictionary-dependent — the algorithm cannot invent `江大桥` unless the name is in the dictionary, which is exactly where OOV enters (§2.4).

### 3.3 Limitations of pure dictionary matching

1. **OOV = guaranteed failure**: anything not in the dictionary cannot be segmented (it either splits into single characters or attaches wrongly). Names and new words are the casualties.
2. **Ambiguity is unresolved**: maximum matching is greedy and local — it cannot see the whole sentence to pick the globally most plausible segmentation, and it cannot handle combination ambiguity at all (`确实` vs `确 / 实` depends on context the matcher never considers).
3. **Dictionary maintenance is unbounded**: Chinese neologisms (新词) appear weekly; a dictionary is forever chasing the language.
4. **Granularity is frozen** in the dictionary's word list.

Pure matching survives today mostly as a *component*: jieba builds its DAG (directed acyclic graph) of all dictionary hits and then applies dynamic programming (§4.2), and the HMM only fires for OOV. Pure FMM/BMM is still taught and still used in embedded/low-resource settings, but no serious system relies on it alone.

---

## 4. Classical Approaches II: Statistical Segmentation (HMM, N-gram, CRF)

The statistical revolution reframed segmentation: instead of *matching words*, **treat it as a tagging or decoding problem over character sequences**, and let probabilities decide between ambiguous readings.

### 4.1 Segmentation as tagging: the BEMS scheme

The key reformulation (Xue, 2003; widely used thereafter): **each character is tagged with its position inside its word**, and word boundaries are implied by the tag sequence. The standard 4-tag scheme:

| Tag | Meaning | 含义 |
|---|---|---|
| **B** | Begin — first character of a multi-char word | 词首 |
| **E** | End — last character of a multi-char word | 词尾 |
| **M** | Middle — interior character(s) of a ≥3-char word | 词中 |
| **S** | Single — a one-character word | 单字词 |

Example: `客户在银行开立账户` → `客/B 户/E 在/S 银/B 行/E 开/B 立/E 账/B 户/E` (see §13.2 for the full worked tags). The word segmentation is then a lossless function of the tag sequence. This turns CWS into a **sequence-labeling problem**, which is what lets HMMs, CRFs, and later neural models attack it with off-the-shelf machinery. (The BEMS scheme survives to this day — modern neural CWS and even some tokenizer training pipelines still use it.)

### 4.2 HMM: Hidden Markov Models and Viterbi

An **HMM** models the tag sequence as a Markov chain: each character *emits* a tag (B/M/E/S) with some probability, and tags transition to tags with some probability. The most probable tag sequence given the characters is found with the **Viterbi algorithm** — dynamic programming over tag paths, i.e., over *all candidate segmentations* at once (the segmentation problem becomes a shortest-path problem). jieba uses exactly this: it builds a DAG of dictionary word candidates, scores them with a word-frequency (unigram) model, and reserves the HMM to *generate candidates for OOV runs* (unknown-character sequences) using character-emission statistics — the `小明` example in §2.4 is the HMM at work.

The **N-gram / language-model view** generalizes this: assign every candidate segmentation a probability under a word-level n-gram model, then pick the maximum-probability segmentation. Word frequencies and bigram statistics come from large segmented corpora; decoding is Viterbi (or beam search for higher-order models). This "most-probable segmentation" framing is the statistical core of many practical systems, and it is why jieba only needs word *frequencies* — the unigram model — to beat naive matching.

### 4.3 CRF: Conditional Random Fields — the classical state of the art

**CRFs** (Lafferty et al., 2001) are the classical apex: a discriminative sequence-labeling model over the BEMS tags, trained to maximize the conditional probability of the tag sequence given the characters. Where the HMM naively multiplies independent emission/transition probabilities, a **linear-chain CRF** learns *feature weights* over rich, overlapping, context-dependent features, including:

- the current character and its identity/type (digit, letter, punctuation, radical),
- character n-grams to the left and right (context window),
- whether the character is in the dictionary, word-frequency statistics,
- character position within a larger span (e.g., a 姓 character starting a name).

With enough features, CRFs resolved most of the ambiguity that dictionaries and HMMs could not, and **CRF-based systems were the state of the art from ~2005 to ~2015** — the LTP (Harbin Institute of Technology) and THULAC (Tsinghua) segmenters are CRF-family systems; the leading SIGHAN bakeoff systems of the 2005–2010 era were CRFs (and structured perceptrons). CRFs still hold the accuracy/interpretability sweet spot for low-resource and CPU-only deployments.

### 4.4 The "classical" landscape, summarized

| Approach | Model | Ambiguity handling | OOV | Speed | Where used |
|---|---|---|---|---|---|
| Dictionary + FMM/BMM/BM | Greedy matching | Weak | None | Extremely fast | Embedded, teaching, legacy |
| Dictionary + DP + frequencies | Unigram/DAG (jieba core) | Good (frequency) | None (dictionary only) | Fast | jieba, search pipelines |
| Dictionary + HMM | HMM/Viterbi for OOV (jieba full) | Good | Good (character stats) | Fast | jieba (default), production Chinese NLP |
| N-gram LM + Viterbi | Word-level n-gram | Strong | Weak | Medium | Early SOTA, some industry |
| CRF | Feature-based sequence labeling | Strong | Good (features) | Medium | LTP, THULAC, classical SOTA |

jieba occupies a unique spot: **dictionary + DAG + frequency DP for known words, HMM for OOV** — a hybrid that remains the most-installed Chinese segmenter in the world (§10.1), even though it is now a decade past the state of the art.

---

## 5. Evaluation: Metrics, Corpora, and the SIGHAN Bakeoffs

### 5.1 Metrics: Precision, Recall, F1

CWS evaluation compares the produced word boundaries against a gold-standard segmentation:

- **Precision (P)**: fraction of produced words that are correct.
- **Recall (R)**: fraction of gold words that were produced.
- **F1**: harmonic mean 2·P·R/(P+R) — the headline number.

Two refinements matter in practice:

- **OOV recall (R_OOV)**: recall *only on words absent from the training dictionary/corpus*. This is the metric that separates jieba-HMM from pure matching and neural models from CRFs — and it is the metric that predicts real-world performance on news/chat text, where OOV is everywhere. Top systems get in-dictionary recall >97% but OOV recall in the 70–80s; the gap between them is the honest measure of generalization.
- **IV recall (R_IV)**: recall on in-vocabulary words. Reporting both IV and OOV recall (as SIGHAN bakeoff papers did) exposes whether a system is merely memorizing its dictionary.

### 5.2 The SIGHAN bakeoffs and the standard corpora

The field's canonical benchmarks came from the **SIGHAN (ACL Special Interest Group for Chinese Language Processing) International Chinese Word Segmentation Bakeoffs** — bakeoff 2003 (Sapporo), **bakeoff 2005** (Jeju Island, results presented at the 4th SIGHAN Workshop at IJCNLP'05), and bakeoff 2006 (Sydney, which added named-entity recognition). The SIGHAN 2005 datasets — known collectively as **icwb2-data** — remain the standard evaluation resource and are freely available for research:

| Corpus | Provider | Script | Domain | Notes |
|---|---|---|---|---|
| **PKU** | Peking University (北京大学) | Simplified | News | The most-used single benchmark |
| **MSR** | Microsoft Research Asia (微软亚洲研究院) | Simplified | News | Widely used; slightly different annotation standard |
| **CityU** | City University of Hong Kong (香港城市大学) | Traditional | News/mixed | Traditional-script benchmark |
| **AS** | Academia Sinica, Taiwan (中央研究院) | Traditional | News | Traditional-script benchmark |

Bakeoff 2005 headline results: the best participating systems scored roughly **F1 95–96** on PKU and **F1 96–97** on MSR (in-domain, closed-track); CRF systems dominated. Those numbers became the bar every later paper cites: classical CRF ~95–96, neural BiLSTM-CRF ~96–97, BERT-based ~97–98 (see §6.4). Two warnings for anyone quoting these numbers today: (1) the corpora are small (~1–2M characters of training text) and in-domain — modern systems on modern data report much higher raw F1, but the *datasets* are the comparability anchor; (2) the traditional-script corpora (CityU, AS) are rarely tested by modern papers, so "SOTA" claims usually mean PKU/MSR only.

### 5.3 Also in the evaluation ecosystem

- **CTB (Chinese Treebank / 中文树库)**: the Penn-style syntactic treebank; its segmentation standard (words, plus bracketed names) is a common modern benchmark, used with its own train/dev/test splits. Penn Chinese Treebank 6/7/8/9 add years.
- **Modern lexical resources**: 北大语料库 (PKU corpus), 人民日报 (People's Daily) annotated data, and the newer **Universal Dependencies** Chinese treebanks (GSD, UD) for cross-lingual-style evaluation.
- **Domain shift**: the recurring finding (e.g., "BERT Meets Chinese Word Segmentation") is that in-domain F1 is high but **cross-domain F1 drops sharply** — a segmenter tuned on news degrades on finance, chat, or legal text. For a bank building Chinese NLP, evaluate on *your* domain, not on PKU news.

---


## 6. Neural Approaches: From BiLSTM-CRF to BERT

### 6.1 The neural reformulation: segmentation = character tagging

The neural era kept the BEMS reformulation (§4.1) and replaced the feature engineering with learned representations. The canonical architecture, standard since ~2016:

**BiLSTM-CRF for CWS**: each character (and optionally its bigram/word features) is embedded; a **bidirectional LSTM** reads the character sequence left-to-right and right-to-left and produces a context vector per position; a linear layer scores the four BEMS tags per position; a **CRF layer** (the same CRF idea as §4.3, now as a trainable output layer) decodes the globally most consistent tag sequence. Training is end-to-end on labeled character-tag pairs.

Why the BiLSTM-CRF worked so well: the bidirectional LSTM captures long-range context (resolving combination ambiguity like `确实` vs `确 / 实`), the CRF layer enforces tag-sequence legality (B must be followed by M/E, never S, etc.), and character-level input sidesteps dictionary coverage entirely. The landmark reference is *"State-of-the-art Chinese Word Segmentation with Bi-LSTMs"* (Ma, Ganchev, Weiss, EMNLP 2018), which showed that a plain BiLSTM-CRF with character bigram features matched or beat every prior feature-engineered system. Subsequent refinements: adversarial/generative pre-training on unlabeled data, word/character lattice models, and multi-task joint training (segmentation + POS + NER in one model — the HanLP and LTP 4.x design, §10.2–10.3).

### 6.2 Pre-trained transformers: BERT-based CWS

The second neural wave replaced the randomly-initialized LSTM with a **pre-trained transformer encoder**. Fine-tune BERT (or RoBERTa/ERNIE/Chinese-BERT variants) on the BEMS tagging task — typically BERT encoder + CRF head (or MLP head) — and CWS becomes a lightweight fine-tuning problem. Pre-training provides the deep character/word knowledge that supervised CWS data (which is scarce: only ~1–2M chars per benchmark corpus) cannot. The pattern "**BERT + CRF, fine-tuned**" is the current state of the art for classical CWS:

- On PKU/MSR, BERT-based CWS reports **F1 ≈ 97–98%**, versus ~96–97 for BiLSTM-CRF and ~95–96 for CRF (approximate, benchmark-dependent; see §16 for verification caveats).
- Crucially, pre-training also improves **cross-domain generalization** (the weakness identified in §5.3) and **OOV recall** — the transformer has seen the characters and their patterns in diverse contexts.

### 6.3 CWS datasets: what modern systems are trained/evaluated on

| Dataset | Source | Notes |
|---|---|---|
| PKU / MSR (SIGHAN 2005) | 北京大学 / 微软亚洲研究院 | The default benchmarks for CWS papers (simplified, news) |
| CityU / AS (SIGHAN 2005) | 香港城市大学 / 台湾中央研究院 | Traditional script; less used in modern papers |
| CTB (Chinese Treebank) | Penn-style treebank | Word segmentation + syntax; modern papers often use CTB splits |
| UD Chinese (GSD) | Universal Dependencies | Modern, cross-lingual-consistent annotation |
| Domain corpora (finance, medicine, law) | In-house / public | The *right* evaluation target for production systems |

### 6.4 Neural vs classical: the honest comparison

| Axis | Classical (CRF, jieba) | Neural (BiLSTM-CRF, BERT) |
|---|---|---|
| In-domain F1 (PKU/MSR) | ~95–96 (CRF SOTA) | ~97–98 (BERT-based) |
| OOV recall | Good (features) | Best (representations + pre-training) |
| Cross-domain robustness | Weak | Much better with pre-training |
| Training data needed | Feature templates + moderate labeled data | Large unlabeled data (pre-training) + small labeled data (fine-tuning) |
| Compute (inference) | CPU, milliseconds | GPU preferred; heavier |
| Interpretability | High (features) | Low (embeddings) |
| Dependencies | Dictionary optional; no big model | Large model, GPU, tokenizer of its own |

The rule of thumb: **if you can run a transformer, neural CWS wins on quality**; if you are on CPU-only edge/embedded, CRF/jieba-class systems are still production-viable. And note the irony: the "tokenizer" a BERT-based CWS uses is itself a subword tokenizer (WordPiece for BERT-Chinese — *character-level*, since Chinese has no spaces to WordPiece-ize) — the two layers of §1 already meet inside the neural CWS model.

---

## 7. The LLM Era: Subword Tokenizers and SentencePiece

### 7.1 What a subword tokenizer is

LLM tokenizers do **not** segment into words. They learn a **vocabulary of subword units** — character strings (and, in byte-level variants, byte sequences) that occur frequently in the training corpus — and encode any text as the longest-available vocabulary pieces. Three algorithms dominate; all three are implemented in the Hugging Face `tokenizers` library and in Google's **SentencePiece**:

| Algorithm | How it builds the vocab | Marker style | Used by |
|---|---|---|---|
| **BPE (Byte Pair Encoding)** | Start from characters; iteratively merge the most frequent adjacent pair into a new token, until the target vocab size | Word pieces prefixed with `Ġ` (space) in GPT-style | GPT-2/3/4, Llama, Qwen, DeepSeek, Baichuan, InternLM |
| **WordPiece** | Like BPE but merges by *likelihood gain* of the language model, not raw frequency | Continuation pieces prefixed with `##` | BERT, ERNIE, many Chinese BERT variants |
| **Unigram (SentencePiece LM)** | Start from a large candidate set; iteratively *remove* the token whose absence hurts the LM likelihood least | None (SentencePiece's own format) | T5, ALBERT, many SentencePiece-trained models |

**SentencePiece** (Google, 2018) deserves special mention: it trains BPE or Unigram **directly on raw text with no pre-tokenization** — no whitespace splitting, no punctuation rules — by treating the input as a raw byte/Unicode stream and marking spaces with a special meta-symbol (`▁`). This "no pre-tokenizer" design is why SentencePiece is the standard for multilingual and Chinese LLMs (Llama-1/2, Qwen1.x, InternLM, and many others train their vocabularies with it): Chinese doesn't need space rules, and SentencePiece doesn't impose any.

**Byte-level BPE** (GPT-2, 2019) is the third pillar: run BPE over **UTF-8 bytes** rather than characters. This gives (a) universal coverage — every Unicode string is representable, nothing is ever "unknown"; (b) an exactly specified vocabulary (256 byte tokens + merges); and (c) robustness to emoji/mixed scripts. The cost: a Chinese character, which is 3 UTF-8 bytes, may be split into byte fragments unless the merge statistics keep its bytes together. Byte-level BPE is the base of GPT-2, GPT-3/4 (tiktoken), Llama-3, DeepSeek, Qwen2+, Baichuan2 — essentially every modern LLM.

### 7.2 How Chinese behaves inside subword tokenizers

The Chinese experience depends entirely on **whether the training corpus was Chinese-heavy and on the vocab size**:

- **Character-level coverage (common in Chinese-trained vocabs)**: the ~3,500–20,000 frequent characters get their own tokens (either directly or as merged byte sequences). Then common **bigrams/words** (银行, 账户, 支付, 结算, 客户) merge into single tokens if the corpus is big enough. The result: Chinese text encodes at **≈1 token per character or better** — Qwen2.5's tokenizer merges 客户, 银行, 账户, 跨境支付 into single tokens, hitting 0.61 tokens/char on a banking sentence (§9, §13).
- **Byte-level fallback (English-centric vocabs)**: a tokenizer trained mostly on English rarely sees Chinese characters as merges; each character is emitted as 1–3 UTF-8 byte tokens. GPT-2's tokenizer scores **2.33 tokens/char** on Chinese text — 42 tokens for an 18-character sentence (§13) — and the "tokens" are semantically meaningless byte fragments. Llama-2 (32K vocab) sat around 1.5–2 tokens/char; its successor **Llama-3 (128K vocab, tiktoken-based, trained on ~30 languages incl. Chinese text) improved to ~0.8 tokens/char** — and even OpenAI's GPT-4o tokenizer (o200k_base) now contains common Chinese words (§9).
- **The "1 char = 1 token" observation**: for Chinese-native LLM tokenizers (Qwen, DeepSeek) this is roughly true — most characters are exactly 1 token, and frequent 2-char words compress to 1. For English-centric tokenizers the honest statement is "1 char = 1–3 tokens, unpredictably."

### 7.3 The vocabulary size trade-off

Vocab size is the single most impactful tokenizer hyperparameter: bigger vocab → shorter sequences (better compression, cheaper inference per token, larger effective context) but bigger embedding/output matrices (more parameters, more memory) and more training data needed to learn rare tokens. Chinese-centric models push vocab size up precisely to capture characters + common words:

- 32K (Llama-2, many 2023 models): poor Chinese efficiency.
- 50K (GPT-2): no Chinese coverage.
- 64K (Baichuan-7B, Yi): reasonable Chinese.
- 92K–125K (InternLM2 92,544; Baichuan2 125,696): solid Chinese coverage.
- 128K–152K (DeepSeek-V3 128K; Qwen2/2.5 151,643 base / 152,064 config; Llama-3 128,256; GPT-4o o200k_base 200,019): the modern sweet spot — common Chinese words are single tokens, sequence lengths shrink.

The practical consequence: **two models with the same context window can fit very different amounts of Chinese text**, and the tokenizer is a first-class design decision for any Chinese LLM (§9).

---

## 8. Chinese LLM Tokenizers: Qwen, DeepSeek, Baichuan, GLM and Friends

All figures below were verified against the models' published tokenizer configs (see §16); where a number is approximate, it is flagged.

### 8.1 The Chinese-native tokenizers (verified vocab sizes)

| Model family | Tokenizer type | Vocab size (verified) | Notes |
|---|---|---|---|
| **Qwen / Qwen2 / Qwen2.5** | Byte-level BPE (tiktoken-style training) | **151,643 base; 152,064 in config** (incl. special tokens) | The reference Chinese-optimized tokenizer; near 1 token/char on Chinese, merges common words (客户, 银行, 账户, 跨境支付 all single tokens); also efficient on English/code. Qwen2 report: vocab ~151,643. The `151643` number also appears as the BOS token id — a frequent source of confusion. |
| **DeepSeek-V2/V3/R1** | Byte-level BPE | **128K extended vocab (config 129,280)** | DeepSeek-V3 report: "byte-level BPE with an extended vocabulary of 128K tokens"; the pretokenizer and training data were "modified to optimize multilingual compression efficiency" (including Chinese). |
| **Baichuan / Baichuan2** | BPE | **125,696** (Baichuan2 config) | Baichuan-7B/13B used 64K; Baichuan2 expanded to ~125.7K for much better Chinese coverage. |
| **InternLM / InternLM2** | Byte-level BPE (SentencePiece lineage) | **92,544** (InternLM2 config) | Llama-style tokenizer trained on a Chinese-heavy corpus. |
| **Yi (01.AI)** | BPE | **64,000** (Yi-34B config) | Llama-2 architecture; tokenizer trained on Chinese+English data. |
| **GLM-130B / ChatGLM** | icetk (SentencePiece-based) | **150,000** (130K text + 20K image tokens) | icetk was THUDM's bilingual/multimodal tokenizer. GLM-4 keeps the icetk lineage (ChatGLM4Tokenizer; added special tokens start ~151,329). |
| **Qwen1.x (Qwen-7B etc.)** | SentencePiece BPE | ~151,851 | Qwen1 used SentencePiece; Qwen2 moved to byte-level BPE/tiktoken style. |

### 8.2 The English-centric tokenizers, measured against Chinese

| Tokenizer | Vocab | Chinese efficiency (this guide's measurements, §13) | Notes |
|---|---|---|---|
| **GPT-2 / r50k_base** | 50,257 | 2.33 tokens/char (worst case) | Byte-level BPE with zero Chinese coverage; every char becomes byte fragments. |
| **cl100k_base (GPT-3.5/GPT-4)** | 100,256 | 1.11 tokens/char | Slightly better but still no real Chinese word tokens; characters split into byte fragments. |
| **o200k_base (GPT-4o)** | 200,019 | 0.72 tokens/char | *Does* contain common Chinese words (客户, 银行, 账户, 支付 as single tokens) — a big improvement. |
| **Llama-2** | 32,000 | ~1.5–2 tokens/char (approx., from literature) | SentencePiece, English-centric. |
| **Llama-3** | 128,256 | 0.78 tokens/char (measured) | tiktoken-based; trained on 30 languages; single-char tokens for most Chinese, some bigrams. |

### 8.3 What "Chinese-native" means in design terms

A **Chinese-optimized vocabulary** is not just "bigger." The design moves that recur across Qwen, DeepSeek, Baichuan2, GLM, InternLM:

1. **Chinese-heavy training corpus** so character and bigram merge statistics reflect the language (2-char words dominate Chinese; vocabularies capture the most frequent ~100K words/bigrams).
2. **Characters as first-class tokens**: the top ~5–20K characters are single tokens, so nearly every char is 1 token (never byte-fragmented).
3. **Common words/bigrams as single tokens**: 银行, 账户, 支付, 结算, 客户… → 1 token each, giving the 0.6–0.8 tokens/char compression (§9).
4. **Balanced multilingual budget**: tokens reserved for English/code (Qwen's English efficiency is also strong), so the model doesn't sacrifice the other half of its workload.
5. **Byte-level fallback** so rare characters, emoji, and mixed text never produce "unknown" tokens.

The economics consequence: **a Chinese-native tokenizer can make Chinese *cheaper* than English** — on a same-meaning banking sentence, Qwen2.5 used 11 Chinese tokens vs 14 English tokens (§13.4). The "Chinese is token-hungry" reputation is a property of *English-centric* tokenizers, not of Chinese.

---

## 9. Token-Efficiency Economics: Is Chinese Token-Hungry?

### 9.1 The claim, and what the data actually shows

The folklore: "Chinese text needs ~1.5–2× the tokens of the equivalent English text, so Chinese costs more and fills the context window faster." **This is true for English-centric tokenizers and false (sometimes inverted) for Chinese-native ones.** Measured on the same banking sentence (18 Chinese chars vs 79 English chars, §13):

| Tokenizer | Chinese tokens | English tokens | Chinese/English ratio | Verdict |
|---|---|---|---|---|
| GPT-2 / r50k_base | 42 | 15 | **2.80×** | Chinese is *very* token-hungry (byte fragmentation) |
| cl100k_base (GPT-4-era) | 20 | 14 | **1.43×** | Moderately hungry |
| o200k_base (GPT-4o) | 13 | 14 | 0.93× | Parity (Chinese words in vocab) |
| Llama-3 (128K, tiktoken) | 14 | 14 | 1.00× | Parity |
| **Qwen2.5 (Chinese-native)** | 11 | 14 | **0.79×** | **Chinese is cheaper** |

Independent data points: a 2025 paired-test study (Mason AI Lab) measured o200k_base using **1.06–1.55× more tokens for Chinese** than English across six sentence pairs — consistent with the o200k row above, and a reminder that the ratio varies with text (dense formal Chinese compresses better than chatty text). English-centric tokenizers on Chinese text are commonly reported at 1.5–2.5× (e.g., an English-centric 50K tokenizer at ~2.5× on short sentences, per a 2024/2025 tokenizer tutorial). The three regimes, in one line:

> **Byte-fragmented (GPT-2/Llama-2): ~2–3 tokens/char. Character-level (Llama-3/o200k): ~0.7–1 token/char, roughly parity with English. Word-merging (Qwen/DeepSeek): ~0.6–0.8 token/char — Chinese can undercut English.**

### 9.2 Why the ratio varies: vocab size × training corpus

Three interacting causes: (1) **vocab size** — bigger vocabularies can afford to store Chinese characters and words (a 32K vocab cannot; a 152K vocab can); (2) **corpus language mix** — tokenizers trained on Chinese-heavy data learn Chinese merges; (3) **byte vs character base** — byte-level BPE fragments characters unless merges hold them together. A character-based (non-byte) tokenizer like a Chinese SentencePiece model is guaranteed ≤1 token/char *by construction*; a byte-level one depends on merge statistics.

### 9.3 The "1 char = 1 token" observation

Users of Qwen/DeepSeek-family models routinely observe that Chinese text costs **about 1 token per character** (and often less, because common 2-char words collapse to 1 token — the measured 0.61 tokens/char in §13). This is the single most practical number for budgeting: for Chinese-native models, budget **≈ 1–1.3 tokens per Chinese character** including punctuation; for English-centric models (Llama-2, GPT-3.5-era, and any 32K–64K vocab model), budget **≈ 1.5–2.5 tokens per character** or switch models.

### 9.4 Mitigations when the tokenizer is English-centric

1. **Pick a Chinese-native model** for Chinese-heavy workloads (Qwen, DeepSeek, GLM, Baichuan, InternLM) — the cheapest mitigation by far.
2. **Use the model's own tokenizer for counting**, never a generic rule — §11.4.
3. **Pre-compress**: chunk by *characters* with overlap for RAG (§11.1), and consider whether common boilerplate can be moved out of the prompt (instructions vs data).
4. **Watch context math**: 32K context with Llama-2-class tokenization holds ~13–16K Chinese chars; the same budget on Qwen2.5 holds ~26K+ chars. RAG chunk sizes and top-k must be sized per tokenizer (§11.1, and see [RAG vs Long-Context LLMs](rag_vs_long_context_llms_guide.md)).
5. **Cost modeling**: LLM pricing is per token (input/output). A 2.8× Chinese overhead on an English-centric API is a 2.8× bill on the Chinese portion of traffic; a Chinese-native model or tokenizer-aware routing removes most of it (§11.3).

### 9.5 Practical impact on context windows and RAG

The context-window implication is direct: **Chinese text fills the window faster, so effective context (in characters) depends on the tokenizer**. For RAG specifically: embedding models (§11.2) are trained on character sequences, retrieval chunk sizes are usually specified in characters for Chinese (256–512 chars ≈ 0.25–0.5K tokens on Chinese-native tokenizers, vs 0.5–1K+ tokens on English-centric ones) — so the *same* retrieval pipeline can silently double its token cost per query depending on the generation model's tokenizer. Cross-references: [Advanced RAG Techniques](advanced_rag_techniques_guide.md) for chunking strategies, [Vector Databases](vector_databases_guide.md) for embedding/retrieval plumbing, [RAG vs Long-Context LLMs](rag_vs_long_context_llms_guide.md) for the context-window trade-off.

---

## 10. Libraries and Tools

### 10.1 jieba (结巴) — the most popular Chinese segmenter in the world

- **What**: pure-Python (with Java/C++/Go ports) segmenter; the default `pip install jieba` answer since 2013.
- **How**: dictionary + DAG + dynamic programming over word frequencies for known words; **HMM + Viterbi for OOV**; modes: `lcut` (precise, default), `lcut_all` (full/所有可能), `lcut_for_search` (fine-grained for search engines), plus custom dictionary loading (`add_word`, `load_userdict`) — the classic way to inject bank-specific terms (同业拆借, 跨境人民币).
- **Status (verified)**: last PyPI release **0.42.1 (September 2020)**; GitHub activity minimal since — effectively **feature-complete and in maintenance mode**, not dead (it still installs cleanly and works), but no new features coming. Its ~350K-entry dictionary is dated; expect to add your own domain terms.
- **Best for**: quick scripts, small services, search-token preprocessing, teaching, and any CPU-only environment. Not competitive on accuracy with neural systems, and its HMM cannot learn from your data.

### 10.2 HanLP — the modern neural toolkit

- **What**: HanLP 2.x (hankcs) — a Java/Python multi-task NLP toolkit: CWS, POS, NER, dependency parsing, semantic role labeling, and more, in one framework, with pre-trained models (including BERT-family backbones).
- **How**: neural models (transformer-based); supports both simplified and traditional Chinese; install with `pip install hanlp` (Python) or use its Java/HTTP interfaces. HanLP 2.1 was trained on open corpora (CTB, PKU, MSR, etc.) with a unified multi-task setup.
- **Status**: actively maintained; the strongest open-source *general* Chinese NLP toolkit behind PaddleNLP.
- **Best for**: production Chinese NLP needing segmentation + NER + POS under one API, with GPU available.

### 10.3 LTP — Language Technology Platform (哈工大语言技术平台)

- **What**: the Harbin Institute of Technology (HIT) platform — one of the most cited Chinese NLP systems ever (LTP 1.x, 2006+; now **LTP 4.x**, PyTorch-native).
- **How**: LTP 4 is a neural pipeline (joint model of segmentation/POS/NER/dependency parsing); ships small fast models and larger accurate ones. `pip install ltp`.
- **Status**: actively maintained (LTP 4.x releases continue); strong academic pedigree (HIT 社会计算与信息检索研究中心).
- **Best for**: Chinese NLP pipelines where a single maintained toolkit for CWS+NER+parsing is wanted; also the canonical CRF-era reference point (LTP 3.x was CRF-based).

### 10.4 THULAC — Tsinghua lexical analyzer (清华大学)

- **What**: Tsinghua University's lexical analyzer (THU Lexical Analyzer for Chinese), C++/Python/Java.
- **How**: classical structured-perceptron/CRF-family model with a large segmentation+POS dictionary; very fast, CPU-friendly.
- **Status**: maintained but quiet; last releases are older.
- **Best for**: fast, dependency-free segmentation + POS on CPU; historical comparison baseline.

### 10.5 PaddleNLP — Baidu's industrial toolkit

- **What**: Baidu's full NLP framework built on PaddlePaddle, home of the **ERNIE** model family (including Chinese ERNIE 3.0/4.0 and the lightweight ERNIE-3.0-tiny used for CWS).
- **How**: provides `LAC` (Lexical Analysis of Chinese — segmentation + POS + NER in one fast model) plus jieba-style APIs and full fine-tuning tooling; `pip install paddlepaddle paddlenlp`.
- **Status**: very actively maintained; industrial-scale documentation and model zoo; the Chinese ecosystem's answer to spaCy-plus-LLM.
- **Best for**: teams standardizing on Baidu's stack; ERNIE-based embedding/classification pipelines; production scale.

### 10.6 The rest of the field

| Library | Type | Approach | Notes |
|---|---|---|---|
| **pkuseg** | Python | Neural (BiLSTM) CWS with domain models | Peking University; strong accuracy on news/medicine/tourism domains; maintenance slowed. |
| **SnowNLP** | Python | Dictionary + rules (simplified Chinese) | Lightweight sentiment-oriented toolkit; segmentation is a side feature; not production-grade accuracy. |
| **FoolNLTK** | Python | BiLSTM | Older neural toolkit; largely unmaintained; kept for completeness. |
| **Stanza** (Stanford) | Python/Java | Neural (BiLSTM + transformer options) | Stanford's multilingual pipeline; Chinese supported; good for cross-lingual consistency (UD-style annotations). |
| **spaCy (zh)** | Python | Statistical | Chinese support via `zh_core_web` (character-based) — weaker word-level CWS out of the box. |

### 10.7 The "modern" stack: tokenizers, not segmenters

For LLM work, the relevant tools are the tokenizer libraries:

- **`tokenizers` (Hugging Face)** — the Rust-backed library behind `AutoTokenizer`; loads BPE/WordPiece/Unigram models from `tokenizer.json`; the standard way to use Qwen/DeepSeek/Llama tokenizers in Python (`transformers.AutoTokenizer.from_pretrained(...)`).
- **`tiktoken` (OpenAI)** — fast Rust BPE encoder for OpenAI models (`cl100k_base`, `o200k_base`) and the encoding engine many open models now adopt (Llama-3's tokenizer is tiktoken-format; Qwen2's training used tiktoken-style BPE).
- **`sentencepiece` (Google)** — trains and runs Unigram/BPE on raw text; used to *train* many Chinese tokenizers; still the right tool if you need a custom tokenizer for a new corpus.
- **`transformers`** — the umbrella: `AutoTokenizer.from_pretrained("Qwen/Qwen2.5-0.5B")` downloads and runs any model's tokenizer without the model weights (§13 uses exactly this).

The one-line distinction to keep: **a segmenter (jieba/HanLP/LTP) gives you words for search/NER/POS; a tokenizer (transformers/tiktoken/sentencepiece) gives you IDs for an LLM.** They are different tools for different layers of the stack (§1, §12.1).

### 10.8 Library comparison table

| Library | Type | Approach | Language | Status | Best for |
|---|---|---|---|---|---|
| jieba | Segmenter | Dict + DP + HMM | Python (ports: Java/C++/Go) | Maintenance mode (0.42.1, 2020) | Quick CPU segmentation, search tokens, teaching |
| HanLP 2.x | Segmenter + NLP suite | Neural (transformers), multi-task | Python/Java | Active | Production CWS+NER+POS on GPU |
| LTP 4.x | Segmenter + NLP suite | Neural (PyTorch) | Python | Active | HIT pipeline; joint segmentation/NER/parsing |
| THULAC | Segmenter + POS | Structured perceptron/CRF | C++/Python/Java | Quiet | Fast CPU segmentation + POS |
| PaddleNLP (LAC/ERNIE) | Segmenter + full framework | Neural (PaddlePaddle) | Python | Very active | Baidu stack; ERNIE pipelines; industrial scale |
| pkuseg | Segmenter | Neural (BiLSTM) | Python | Slowed | Domain-specific CWS accuracy |
| SnowNLP | Segmenter + sentiment | Dictionary/rules | Python | Slow | Lightweight experiments |
| Stanza | Segmenter + NLP suite | Neural | Python/Java | Active | Stanford/UD-consistent multilingual pipelines |
| tokenizers / transformers | Tokenizer | BPE/WordPiece/Unigram | Python/Rust | Very active | Loading/running LLM tokenizers |
| tiktoken | Tokenizer | Byte-level BPE | Python/Rust | Very active | OpenAI & tiktoken-format tokenizers, fast counting |
| sentencepiece | Tokenizer *trainer* | Unigram/BPE on raw text | C++/Python | Active | Training custom tokenizers for new corpora |

---


## 11. Chinese Tokenization in LLM Applications (RAG, Cost, Fine-Tuning)

### 11.1 Chinese RAG: chunking and retrieval

Chinese RAG inherits every decision from the English playbook (see the [RAG series](rag_frameworks_comparison_guide.md) and [Vector Databases](vector_databases_guide.md)) plus three Chinese-specific ones:

1. **Chunk by characters, not words.** The de facto convention for Chinese chunking is **fixed-size character windows with overlap** (e.g., 256–512 characters, 10–20% overlap, splitting on sentence-ending punctuation 。！？ when possible). Word-based chunking (jieba then chunk words) is tempting but fragile: segmentation errors become chunking errors, and the embedding model was usually trained on raw characters anyway. Character-window chunking is tokenizer-agnostic and trivially reproducible.
2. **Choose chunk size from the *generation* model's tokenizer.** The retrieval side doesn't care (the embedding model sees characters), but the prompt budget does: 512 chars ≈ 312 tokens on Qwen2.5 (0.61/char) but ≈ 1,200 tokens on GPT-2-class byte tokenizers. If you size chunks for one model and switch models, re-check the math (§9.5). For long-context models the calculus changes — see [RAG vs Long-Context LLMs](rag_vs_long_context_llms_guide.md).
3. **Segmenting the query for hybrid search.** Traditional + lexical hybrid retrieval (BM25 over jieba-segmented text) still beats pure vector search on exact terms (account numbers, product names, regulation citations) — see [BM25 / FAISS / ScaNN Research](bm25_faiss_scann_research.md). jieba's `lcut_for_search` mode exists precisely for this: index coarse and fine units so the inverted index matches both 清华大学 and 清华. Word-level segmentation is the *one* place classical CWS still earns its keep in the LLM stack.

### 11.2 Chinese embedding models

Retrieval quality for Chinese depends on the embedding model's Chinese training, not on segmentation:

- **BGE (BAAI)**: `bge-large-zh-v1.5`, `bge-base-zh-v1.5`, and the multilingual `bge-m3` — the de facto standard open Chinese/multilingual embedders; BGE-M3 also does sparse/dense hybrid embeddings and long inputs.
- **text2vec (shibing624)**: `text2vec-large-chinese` and family — a popular open Chinese sentence-embedding line.
- **ERNIE-embedding (Baidu)**, **gte (Alibaba)**, **M3E (moka-ai)** — additional Chinese embedding options in the same ecosystem.
- **OpenAI/other multilingual APIs** (text-embedding-3, voyage) — acceptable but usually behind Chinese-native models on Chinese similarity benchmarks.

Rule of thumb: for Chinese RAG, prefer a Chinese/multilingual embedding model and **do not segment before embedding** (character-level input is what these models were trained on).

### 11.3 LLM cost in Chinese applications

Per-token pricing (input/output) makes tokenizer efficiency a direct cost line:

- On a Chinese-native model at ~0.6–1 token/char, a 1,000-char Chinese document costs ~600–1,000 input tokens.
- On an English-centric byte-level tokenizer at ~2.3 tokens/char, the same document costs ~2,300 tokens — a 2–4× cost difference *before* any model-quality differences.
- Chinese applications should therefore (a) prefer Chinese-native models for Chinese-heavy traffic, (b) count tokens with the *serving* model's tokenizer (§11.4), and (c) price Chinese workloads in the business case using measured per-character token rates, not English rules of thumb. For on-prem deployment, tokenizer efficiency also translates into KV-cache and context memory — see [On-Prem LLM Deployment](../on_prem_llm_deployment_guide.md) and [LLM Optimization Complete Guide](llm_optimization_complete_guide.md).

### 11.4 Tokenizer-aware design

- **Token counting**: never estimate Chinese tokens with English heuristics ("1 token ≈ 4 chars"). Use the model's real tokenizer: `tiktoken` for OpenAI models; `AutoTokenizer.from_pretrained(model_id)` for open models (Qwen, DeepSeek, Llama). Count before every API call when enforcing prompt budgets; the same text can differ 3× across tokenizers (§13).
- **Prompt design**: keep instructions/data separation tight; Chinese boilerplate (system prompts, few-shot examples) consumes the budget fast on byte-level tokenizers — on Chinese-native models it is comparatively cheap. If you must support a byte-fragmented tokenizer, minimize Chinese few-shot examples and prefer concise formulations.
- **Context management**: track context usage in characters AND tokens; evict or summarize oldest content by measured token count, not message count (see [Agent Runtime Cache Design](agent_runtime_cache_design_guide.md) and [Beyond RAG — Agent Memory](beyond_rag_guide.md)).

### 11.5 Fine-tuning Chinese models

- **The tokenizer is frozen**: standard fine-tuning (LoRA/QLoRA/full) does not retrain the tokenizer. If the base tokenizer byte-fragments Chinese, fine-tuning inherits the inefficiency; you cannot "fix" it without retraining the vocabulary + embeddings (a serious undertaking).
- **Adding domain terms**: for open models you *can* add tokens (`tokenizer.add_tokens([...])` + `model.resize_token_embeddings`) for a small set of high-value bank terms (同业拆借, 跨境人民币, 掉期交易) before fine-tuning — a well-known trick, with the caveat that new tokens start unembedded and need enough exposure to be learned. Do this for 10–100 terms, not thousands.
- **Choosing a base**: for Chinese fine-tuning, start from a Chinese-native base (Qwen, DeepSeek, GLM, Yi, Baichuan, InternLM) — you inherit the efficient tokenizer and the Chinese pre-training. See [Fine-Tuning Frameworks Comparison](fine_tuning_frameworks_comparison_guide.md).

### 11.6 Chinese in code

Tokenizers treat code and Chinese independently: Chinese **string literals** inside code are tokenized as Chinese (same efficiency story as prose), while Chinese **identifiers** are extremely rare in practice (almost all Chinese code uses ASCII identifiers with Chinese comments — comments tokenize as Chinese text). The practical implication for code models: Chinese comments/strings inflate token counts on English-centric code tokenizers; Chinese-native models (Qwen2.5-Coder, DeepSeek-Coder) handle both well.

---

## 12. Practical Guidance

### 12.1 When to segment (words) vs when to tokenize (IDs)

| Task | Layer | Tool | Why |
|---|---|---|---|
| Lexical/BM25 search over Chinese docs | Segment (words) | jieba `lcut_for_search`, HanLP | Inverted indexes need word units; fine+coarse units maximize recall |
| NER / POS / dependency parsing | Segment (words) | HanLP, LTP, PaddleNLP LAC | Classical NLP models are trained on word/character-tag corpora |
| Keyword extraction, text statistics | Segment (words) | jieba, THULAC | Word frequency is the unit of analysis |
| LLM input/output encoding | Tokenize (IDs) | transformers / tiktoken / sentencepiece | The model defines its own vocabulary; words are irrelevant to it |
| Embedding / vector retrieval | Neither (raw chars) | BGE/text2vec embedders | Chinese embedding models are character-trained; segmenting hurts |
| RAG chunking | Neither (char windows) | Custom chunker | Character windows are tokenizer-agnostic and reproducible |

### 12.2 Library selection

- **Simple / quick / CPU-only**: jieba (with `add_word` for your domain dictionary). Fast, ubiquitous, good enough for search preprocessing.
- **Modern production Chinese NLP (segmentation + NER + POS)**: HanLP 2.x or LTP 4.x (GPU); PaddleNLP LAC if you're on the Baidu stack.
- **LLM work**: the model's own tokenizer via `transformers`/`tiktoken`; `sentencepiece` only if you must *train* a custom tokenizer.
- **Domain-heavy text (banking)**: whatever segmenter you choose, load a domain user-dictionary (jieba `load_userdict`, HanLP `dict` customization) with your institution's terms — this single step fixes most production segmentation complaints.

### 12.3 Token-efficiency best practices

1. Choose Chinese-native models for Chinese-heavy workloads (biggest lever).
2. Measure, don't guess: run your real corpus through the serving tokenizer once and record tokens/char for Chinese, English, and mixed text (§13.6 has the code).
3. Size RAG chunks and context budgets in the serving model's tokenizer (§11.1, §9.5).
4. Keep system prompts lean; move static Chinese instructions into templates outside the per-request budget where possible.
5. For cost reporting, quote Chinese token rates per 1,000 chars (e.g., "Qwen2.5 ≈ 610 tokens / 1,000 chars") so finance can model volume-based cost.

### 12.4 Pitfalls

- **Segmentation errors cascade**: in word-based pipelines (search, NER), one bad boundary (南京市长江大桥 → 南京/市长/江大桥) corrupts downstream features. Mitigate with domain dictionaries and, for production, neural segmenters.
- **OOV in production**: names, new terms, transliterations (麦当劳, 特朗普, 跨境人民币). Always evaluate OOV recall on *your* domain; add words to the dictionary rather than fighting the segmenter.
- **Mixed Chinese–English (中英混合 / code-switching)**: real text mixes scripts — 使用API调用, "OpenAI的GPT-4模型", 客户ID. Chinese-native tokenizers handle this well (their vocab covers both); byte-fragmented tokenizers degrade *both* halves. For search, consider segmenting Chinese and English separately (whitespace-split the ASCII, jieba the Chinese) before indexing.
- **Simplified vs traditional (简繁)**: 客户 (simplified) vs 客戶 (traditional) are different characters/tokens. Pick one standard per system; convert with OpenCC if both must be supported; be aware that some corpora (CityU, AS) are traditional.
- **Punctuation and numbers**: full-width （）？。 vs half-width ()?. and Arabic numerals mixed into Chinese text — tokenizers and segmenters treat them differently; normalize early.
- **Don't trust "1 token ≈ 4 characters"**: that English heuristic is off by 2–4× for Chinese depending on the tokenizer (§9).

### 12.5 Evaluating tokenization

For **CWS quality**: precision/recall/F1 against a gold standard (PKU/MSR/CTB or your own annotated sample), always split out OOV recall; test cross-domain (news-trained vs your finance text). For **tokenizer efficiency**: tokens-per-character on representative Chinese, English, and mixed samples; compression ratio vs a reference tokenizer; effective context in characters. For **production**: measure end-to-end (retrieval recall, NER F1, cost per 1,000 queries), not just segmentation F1.

---

## 13. Worked Example: One Sentence, Five Tokenizers

All outputs below are **real, measured today** (August 2026) — jieba 0.42.1, transformers 5.x, tiktoken, and the actual Qwen2.5 / Llama-3 tokenizers downloaded from Hugging Face. The sentence is a banking one: **`客户在银行开立账户用于跨境支付结算。`** ("The customer opened an account at the bank for cross-border payment settlement.")

### 13.1 Step 1 — jieba word segmentation (classical CWS)

```python
import jieba
s = "客户在银行开立账户用于跨境支付结算。"
print(" / ".join(jieba.lcut(s)))            # precise (default)
print(" / ".join(jieba.lcut_for_search(s))) # fine-grained, for search indexes
```

Output (measured):

```
客户 / 在 / 银行 / 开立 / 账户 / 用于 / 跨境 / 支付 / 结算 / 。
客户 / 在 / 银行 / 开立 / 账户 / 用于 / 跨境 / 支付 / 结算 / 。
```

(The search mode matches the precise mode here because every word is already a dictionary word at the fine level.) The ambiguity classic, also measured:

```
南京市长江大桥  ->  南京市 / 长江大桥          (jieba picks the bridge reading)
研究生命起源    ->  研究 / 生命 / 起源
乒乓球拍卖完了  ->  乒乓球 / 拍卖 / 完 / 了      (vs the alternative 乒乓球拍/卖完/了)
```

And jieba's OOV/name handling (HMM on vs off):

```
小明硕士毕业于中国科学院计算所，后在日本京都大学深造
HMM on :  小明 / 硕士 / 毕业 / 于 / 中国科学院 / 计算所 / ， / 后 / 在 / 日本京都大学 / 深造
HMM off:  小 / 明 / 硕士 / 毕业 / 于 / 中国科学院 / 计算所 / ， / 后 / 在 / 日本京都大学 / 深造
```

`小明` and `计算所` are not in jieba's dictionary — the HMM infers them from character statistics. Turn the HMM off and they fragment. This is the OOV problem (§2.4) made visible.

### 13.2 Step 2 — BEMS character tagging

The same segmentation expressed as character tags (the sequence-labeling formulation of §4.1):

```
客户在银行开立账户用于跨境支付结算。
B E S B E B E B E B E B E B E B E S
客/B 户/E 在/S 银/B 行/E 开/B 立/E 账/B 户/E 用/B 于/E 跨/B 境/E 支/B 付/E 结/B 算/E 。/S
```

Every multi-character word contributes B…E; single characters are S. Any CWS model — CRF, BiLSTM-CRF, or BERT+CRF — is trained to predict exactly this tag sequence from the character sequence.

### 13.3 Step 3 — LLM subword tokenization

```python
from transformers import AutoTokenizer
qwen  = AutoTokenizer.from_pretrained("Qwen/Qwen2.5-0.5B-Instruct")
llama = AutoTokenizer.from_pretrained("NousResearch/Meta-Llama-3-8B")  # tiktoken-based
for name, tok in [("Qwen2.5", qwen), ("Llama-3", llama)]:
    ids = tok.encode("客户在银行开立账户用于跨境支付结算。", add_special_tokens=False)
    print(name, len(ids), ids)
```

Measured token IDs:

```
Qwen2.5 (vocab 151,643): 11 tokens
  [100017, 18493, 100358, 29767, 79095, 104827, 100751, 103050, 68262, 106331, 1773]
  decoded pieces: 客户 | 在 | 银行 | 开 | 立 | 账户 | 用于 | 跨境 | 支付 | 结算 | 。

Llama-3 (vocab 128,256): 14 tokens
  [114105, 19000, 112960, 30867, 80195, 81201, 17982, 114593, 123315, 103496, 69362, 37985, 70203, 1811]
  decoded pieces: 客户 | 在 | 银行 | 开 | 立 | 账 | 户 | 用于 | 跨 | 境 | 支付 | 结 | 算 | 。
```

Read the difference: **Qwen2.5 merges the 2-character words** (账户, 跨境, 支付, 结算, 客户, 银行 all single tokens); **Llama-3 splits several of them into single characters** (账户→账+户, 跨境→跨+境, 结算→结+算). Both are far better than a byte-fragmented tokenizer — which, for the same sentence, produces *42* meaningless byte-fragment tokens (GPT-2-style, §13.4). Also note the ambiguity sentence: Qwen2.5 encodes 南京市长江大桥 as 4 tokens (南京市|长江|大桥) — its vocabulary has already "learned" the segmentation that jieba had to compute.

### 13.4 Step 4 — token counts: Chinese vs English, five tokenizers

Same meaning, two sentences: Chinese `客户在银行开立账户用于跨境支付结算。` (18 chars) vs English "The customer opened an account at the bank for cross-border payment settlement." (79 chars). **All numbers measured**:

| Tokenizer | Vocab | Chinese tokens | English tokens | Tokens/char (zh) | zh/en ratio |
|---|---|---|---|---|---|
| GPT-2 / r50k_base (byte-level BPE) | 50,257 | **42** | 15 | 2.33 | **2.80×** |
| cl100k_base (GPT-3.5/GPT-4) | 100,256 | 20 | 14 | 1.11 | 1.43× |
| o200k_base (GPT-4o) | 200,019 | 13 | 14 | 0.72 | 0.93× |
| Llama-3 (tiktoken-based) | 128,256 | 14 | 14 | 0.78 | 1.00× |
| **Qwen2.5 (Chinese-native)** | 151,643 | **11** | 14 | **0.61** | **0.79×** |

Takeaways, all directly evidenced:

- The folklore "Chinese ≈ 1.5–2× English tokens" is **true only for the byte-fragmented/English-centric end** of the table (GPT-2: 2.8×; cl100k: 1.43×).
- **Chinese-native and modern multilingual tokenizers reach parity or better** (Llama-3: 1.0×; o200k: 0.93×; Qwen2.5: 0.79× — Chinese *cheaper* than English).
- Tokens/char spans **0.61 → 2.33** across tokenizers for the *same text* — a 3.8× spread. Never use a universal token-per-char heuristic (§12.4).
- GPT-4o's o200k_base genuinely contains common Chinese words (客户, 银行, 账户, 支付 each encode as one token) — a marked improvement over cl100k_base (GPT-4-era), which fragmented characters into 2-byte pieces.

### 13.5 Step 5 — the comparison table (char vs word vs subword)

| Granularity | Unit | 客户在银行开立账户… token count | Pros | Cons |
|---|---|---|---|---|
| Character (字) | 1 char = 1 unit | 18 | Universal, no segmentation errors | Long sequences; no word semantics |
| Word (词, jieba) | dictionary words | 10 | Matches human intuition; good for search | OOV errors; needs dictionary |
| Subword (Qwen2.5) | learned pieces | 11 | Compact; model-native; no OOV | Opaque units; model-specific |
| Bytes (GPT-2-style) | UTF-8 bytes | 42 | Universal coverage | Extremely inefficient for Chinese |

### 13.6 The code sketches (copy-paste)

```python
# 1) Classical word segmentation — jieba
import jieba
jieba.add_word("跨境人民币")                    # inject bank-specific term
words = jieba.lcut("客户在银行开立账户用于跨境支付结算。")

# 2) BEMS tagging from any segmentation
def bems(words):
    out = []
    for w in words:
        if len(w) == 1: out.append((w, "S"))
        else:
            out.append((w[0], "B"))
            out += [(c, "M") for c in w[1:-1]]
            out.append((w[-1], "E"))
    return out

# 3) LLM tokenization — transformers (Qwen / DeepSeek / Llama / GLM…)
from transformers import AutoTokenizer
tok = AutoTokenizer.from_pretrained("Qwen/Qwen2.5-7B-Instruct")
ids = tok.encode("客户在银行开立账户", add_special_tokens=False)
print(len(ids), ids)                       # real token count for budgeting

# 4) LLM tokenization — tiktoken (OpenAI models)
import tiktoken
enc = tiktoken.get_encoding("o200k_base")  # GPT-4o family; cl100k_base = GPT-4-era
print(len(enc.encode("客户在银行开立账户")))

# 5) Token efficiency report for your corpus
def report(tokenizer, texts):
    for label, t in texts.items():
        n = len(tokenizer.encode(t, add_special_tokens=False))
        print(f"{label}: {n} tokens, {n/len(t):.2f}/char")
```

---

## 14. The Future (2026 and Beyond)

### 14.1 Tokenizer innovation keeps moving

- **Chinese-optimized vocabularies are now table stakes** for Chinese LLMs (Qwen, DeepSeek, GLM, Baichuan, InternLM all ship 90K–152K Chinese-heavy vocabularies). The next frontier is *even better* multilingual compression: DeepSeek-V3 explicitly retuned its pretokenizer and training data "to optimize multilingual compression efficiency"; Llama-3's tiktoken-based 128K tokenizer brought Chinese to parity with English — a decade after Llama-2's 32K tokenizer made Chinese the most expensive major language.
- **Byte-level and multi-lingual vocabularies**: byte-level BPE is now universal (no unknown tokens anywhere), and vocabulary expansion/adapter research (e.g., writing-system-specific tokenizer adaptation, Chinese/Japanese/Korean subword sharing) is an active area — tokenizer surgery on top of frozen models.
- **Token compression in the model**: the DeepSeek-V3 **Multi-Token Prediction (MTP)** head — predicting several tokens per step — reduces the *effective* cost of long Chinese sequences by amortizing per-token overhead, a direction several labs are following.

### 14.2 Longer contexts — does token efficiency matter less?

Context windows grew 4K → 32K → 128K → 1M+ (Gemini-class, and open models reaching 128K–1M) — see [RAG vs Long-Context LLMs](rag_vs_long_context_llms_guide.md). Two countervailing forces keep token efficiency relevant: (1) **cost and latency scale per token** regardless of window size — a 2.8× Chinese overhead is a 2.8× bill and a 2.8× slower generation even with a 1M window; (2) **attention complexity** still grows with sequence length, so shorter sequences (better compression) directly buy inference speed and KV-cache memory. The conclusion: longer windows *relax* the context-fit constraint but *not* the economics; token efficiency remains a first-order design variable.

### 14.3 Tokenizer-free models: byte-level and byte-latent

The most interesting long-term direction is **removing the tokenizer entirely**:

- **MambaByte** (Gu & Dao, 2023): a pure byte-level language model (no vocabulary at all) trained on raw bytes — proved byte-level training feasible with linear-time state-space models; costs more compute per byte but eliminates every tokenization artifact.
- **Byte-latent transformer (BLT, Meta, 2024)**: patches bytes dynamically — groups bytes into variable-length "patches" by a lightweight entropy model, giving tokenizer-free operation with better compute efficiency than MambaByte.
- These are research-stage; every production LLM in 2026 still has a vocabulary. But if byte-patch models scale, the entire Chinese token-efficiency discussion (§9) — and the "which tokenizer" question — becomes moot. For now, the practical stance: **watch BLT-class work; optimize within the tokenizer paradigm.**

### 14.4 Multimodal Chinese

Chinese multimodal models tokenize images/video separately (visual tokens) while text follows the same Chinese-aware tokenizer — e.g., GLM's icetk lineage *reserved* 20K image tokens in its 150K vocabulary from the start, and Qwen-VL/DeepSeek-VL keep the efficient Chinese text tokenizer. For document-heavy banking AI (scanned contracts, ID cards, statements), the OCR→text→Chinese-tokenizer pipeline is where the economics of §9 land.

### 14.5 Trends summary

1. Chinese token efficiency went from worst-in-class (Llama-2) to best-in-class (Qwen/DeepSeek) in ~18 months — vocabulary design is now a competitive weapon.
2. Classical CWS is not dying; it relocated to search/lexical retrieval, NER, and domain dictionaries while LLMs absorbed its output layer (LLM vocabularies encode words as tokens).
3. Tokenizer-free byte models are the credible 2030-era endgame; meanwhile, tokenizer-aware engineering (counting, chunking, model choice) is the highest-ROI skill for Chinese LLM applications.
4. For banks: the practical 2026 playbook is Chinese-native models + character-window RAG + measured token budgets + domain dictionaries — everything in this guide's §11–§13.

---

## 15. Glossary

| Term | 中文 | Definition |
|---|---|---|
| CWS (Chinese Word Segmentation) | 中文分词 | The NLP task of splitting unsegmented Chinese text into words |
| Character | 字 | A single Chinese ideograph/syllable (e.g., 银) |
| Word | 词 | One or more characters forming a lexical unit (e.g., 银行) |
| Segmentation ambiguity | 歧义 | A character string with multiple valid segmentations (交集型/组合型) |
| Overlap ambiguity | 交集型歧义 | A substring competing between adjacent words (南京市长江大桥) |
| Combination ambiguity | 组合型歧义 | A string that is a word AND a sequence of smaller words (确实 vs 确/实) |
| OOV (out-of-vocabulary) | 未登录词 | Words absent from the dictionary/training data: names, neologisms, transliterations |
| FMM / BMM / BM | 正向/逆向/双向最大匹配 | Greedy dictionary matching, forward, backward, or both |
| HMM | 隐马尔可夫模型 | Statistical tag-sequence model; used with Viterbi for segmentation/OOV |
| BEMS | 词首/词尾/词中/单字 | The 4-tag character-position scheme: Begin, End, Middle, Single |
| N-gram / language model | n元语法/语言模型 | Word/char frequency statistics scoring candidate segmentations |
| CRF | 条件随机场 | Feature-based sequence labeling model; classical CWS SOTA (~95–96 F1) |
| BiLSTM-CRF | — | Neural CWS: bidirectional LSTM + CRF output layer (~96–97 F1) |
| BERT-based CWS | — | Pre-trained transformer fine-tuned for BEMS tagging (~97–98 F1) |
| SIGHAN Bakeoff | 国际中文处理评测 | ACL's Chinese-NLP bakeoffs; SIGHAN 2005 defined the standard CWS benchmarks |
| PKU / MSR / CityU / AS | 北大/微软/城大/中研院 | The four SIGHAN 2005 corpora (simplified ×2, traditional ×2) |
| CTB | 中文树库 | Penn Chinese Treebank; word segmentation + syntax benchmark |
| Subword tokenization | 子词切分 | Encoding text as learned subword vocabulary units, not words |
| BPE | 字节对编码 | Merge most frequent adjacent pairs until the target vocab size |
| Byte-level BPE | 字节级BPE | BPE over UTF-8 bytes (GPT-2 style); universal coverage, multi-byte chars |
| WordPiece | — | BERT-style likelihood-gain merging; `##` continuation markers |
| Unigram | 一元模型分词 | SentencePiece LM algorithm: prune tokens by likelihood impact |
| SentencePiece | — | Google's raw-text tokenizer trainer (no pre-tokenization; `▁` space marker) |
| Vocabulary / vocab size | 词表/词表大小 | The token set of a tokenizer; 32K–200K typical for LLMs |
| Qwen / DeepSeek / Baichuan / GLM / Yi / InternLM | 通义/深度求索/百川/智谱/零一/书生 | Chinese LLM families with Chinese-optimized tokenizers (64K–152K) |
| jieba / HanLP / LTP / THULAC / PaddleNLP | 结巴/汉语言平台/哈工大平台/清华词法/百度NLP | Chinese segmentation/NLP libraries |
| tiktoken / transformers / tokenizers | — | Tokenizer libraries: OpenAI BPE; Hugging Face model tokenizers |
| Token efficiency | token效率 | Tokens per character/word; the driver of cost and context usage |
| Token overhead | token开销 | Extra tokens a tokenizer spends on a language vs a reference |
| Context window | 上下文窗口 | Max tokens a model can ingest; fills faster for byte-fragmented Chinese |
| RAG chunking | 检索增强分块 | Splitting documents for retrieval; character windows for Chinese |
| BGE / text2vec | — | Chinese/multilingual embedding model families |
| ERNIE | 文心 | Baidu's pre-trained model family (PaddleNLP ecosystem) |
| Tokenizer-free | 免分词器 | Byte-level/byte-patch models (MambaByte, BLT) with no vocabulary |

---

## 16. Verification Notes & Sources

**Measured in this session (August 2026), not estimated:**

- All token counts, token IDs, tokens/char ratios, and zh/en ratios in §9 and §13 — run with jieba 0.42.1, transformers 5.x, tiktoken, and the actual `Qwen/Qwen2.5-0.5B-Instruct` and `NousResearch/Meta-Llama-3-8B` tokenizers.
- jieba's segmentation of 南京市长江大桥, 研究生命起源, 乒乓球拍卖完了, and the HMM on/off OOV demo (§13.1).
- Vocab sizes from published model configs: Qwen2/Qwen2.5 `config.json` (152,064; tokenizer 151,643), Baichuan2-7B (125,696), InternLM2-7B (92,544), Yi-34B (64,000), Llama-3 (bos 128,000, vocab 128,256), GLM-4 `tokenizer_config.json` (ChatGLM4Tokenizer, icetk lineage).
- DeepSeek-V3: byte-level BPE, 128K extended vocabulary (technical report), HF config vocab_size 129,280.
- jieba last PyPI release 0.42.1 (September 2020) — from PyPI.

**External sources consulted:** SIGHAN 2005 bakeoff site (sighan.cs.uchicago.edu/bakeoff2005) and icwb2-data (corpora: AS, CityU, PKU, MSR); DeepSeek-V3 technical report (arXiv 2412.19437); Mason AI Lab token-efficiency paired tests (o200k_base: 1.06–1.55× more Chinese tokens); Qwen2 report (byte-level BPE, ~151,643 vocab); Ma et al., "State-of-the-art Chinese Word Segmentation with Bi-LSTMs" (EMNLP 2018); "BERT Meets Chinese Word Segmentation" (generalization findings); GLM-130B icetk documentation (150,000 vocab: 130K text + 20K image); PyPI/GitHub pages for jieba, HanLP, LTP, THULAC, PaddleNLP.

**Flagged as approximate (not independently verified):** Llama-2 Chinese efficiency (~1.5–2 tokens/char, from secondary literature); Qwen1.x vocab size (151,851, from secondary sources); Baichuan-7B original 64K vocab (from secondary sources); classical vs neural F1 ranges in §6.4 (paper-reported ranges, benchmark-dependent); the "~350K entries" jieba dictionary size (widely cited, not re-counted here); BGE/text2vec model availability (well-established, not re-checked against the hub today). Treat these as directional.

**Corrections welcome:** open an issue/PR on [github.com/jackliusr/research](https://github.com/jackliusr/research) if any number above is stale — tokenizer vocabularies change with every model release.
