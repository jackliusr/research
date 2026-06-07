
## Prompt for hermes agent
```
Research how to accelerate secondary language learning by combining below technologies using autoresearch
. spaced repetition
. generate multi-modal content using llm models based on words needed to review
. exercises or quizzes to help make passive vocabulary to active vacabulay
```



## Research Results: LLM + SRS + Active Recall for Language Learning

1. 🧠 The Three Pillars — What We Know

A. Spaced Repetition (SRS)
- State of the art: FSRS (Free Spaced Repetition Scheduler) — now built into Anki, ML-based, per-card difficulty adaptation, 30–50% fewer reviews for same retention vs. SM-2. Python library available (fsrs4anki).
- The gap in current SRS: They strengthen the recognition path (form → meaning) but do almost nothing for the production path (meaning → form). That's why learners have 3–5× more passive than active vocab.

B. LLM-Generated Multimodal Content
- What works: LLMs can generate on-demand: example sentences at CEFR-leveled difficulty, short stories using N target words, cloze tests with distractors, mnemonics, image descriptions (for DALL-E / Stable Diffusion), TTS audio (ElevenLabs / OpenAI TTS).
- Key projects: AnkiBrain (most mature, text-only), AI-Flashcards (adds Stability AI images), LinguaLearn (full-stack SRS + GPT). None do the full multimodal + SRS + quiz pipeline.
- Prompt patterns exist for each content type — I found tested templates for difficulty-calibrated sentences, cloze generation with distractors, multi-word story generation, and adaptive difficulty.

C. Passive → Active Vocabulary Exercises
- What actually works (ranked by effectiveness):
  1. L1→L2 translation (strongest retrieval practice — forces word-form recall from meaning)
  2. Cloze deletion with forced typing (not just show-answer)
  3. Sentence completion (tests collocations and grammatical frames)
  4. Free-writing prompts using target words
  5. Error correction (detect and fix misused words)
- Key insight: You need 5–7 productive retrievals per word across different contexts to transition it from passive to active.

2. 🔗 How They Fit Together

The magic is in the feedback loop:

```
SRS Scheduler (FSRS) → Due Word Queue
    ↓
LLM Content Generator (batched, async)
    ├── Text: sentences, stories, cloze, mnemonics
    ├── Image: DALL-E / Stable Diffusion per word
    └── Audio: TTS pronunciation per word + sentence
    ↓
Quiz Engine (interleaved exercise types)
    ├── Session: cloze → translation → completion → free-write
    └── Difficulty adapts based on past errors
    ↓
Review Log → feeds back to FSRS (stability, difficulty)
```

3. 🏗️ Architectural Blueprint

**Scheduler**
• What: FSRS algorithm
• Recommendation: fsrs4anki Python lib — best retention/review ratio

**LLM Text**
• What: Content generation
• Recommendation: Claude / GPT-4o for quality; batch 10 words per call

**TTS Audio**
• What: Pronunciation
• Recommendation: Edge TTS (free, good) or ElevenLabs (premium)

**Image Gen**
• What: Visual mnemonics
• Recommendation: For concrete words only; Stable Diffusion XL (self-host) cheaper than DALL-E

**Quiz Formats**
• What: Exercise types
• Recommendation: Cloze → L1→L2 translation → free-write → error correction, interleaved per word

**Delivery**
• What: User interface
• Recommendation: Telegram bot (you already have the platform) or web app

**Caching**
• What: Cost optimization
• Recommendation: Hash (word + level + L1) → cache audio/images once, regenerate text for variety

4. 🎯 Key Gaps (Opportunities)

No existing tool does multimodal SRS + active recall end-to-end
• Your System Advantage: You'd be first to close the loop

Nobody uses learner performance data to tune LLM prompt difficulty
• Your System Advantage: Feed review history back into prompts — "this user struggled with 'nonetheless', generate easier sentences next time"

Most projects are Anki plugins or single-use scripts****
• Your System Advantage: A standalone Telegram bot with native SRS + LLM generation is novel

**Free-write + error correction at SRS cadence is unexplored** (1/2)

• Your System Advantage: The most valuable exercise type for active vocab, barely automated anywhere

5. 📊 Quick Reality Check

Cost estimate per active daily learner: (1/2) (2/2)

(Claude Haiku / GPT-4o-mini)
- TTS audio: ~$0.0005 per word (Edge TTS = free)
- Image gen: ~$0.002 per image (Stable Diffusion self-host = near-free)
- Total: ~$0.003–0.01 per learner per day for a 20-word session

Implementation order (recommended):
1. FSRS scheduler + SM-2 basics (core loop)
2. LLM text generation for cloze + translation exercises
3. TTS audio playback
4. Active recall quiz engine (interleaved types)
5. Free-write + LLM error correction
6. Image generation (lower priority, higher cost)

That's the full picture. Bottom line: Nobody has built the full multimodal + SRS + active recall pipeline yet — what you're describing is genuinely unclaimed territory. The smartest starting point would be a Telegram bot (your existing platform) with an FSRS scheduler, a Claude-driven content generator, and interleaved quiz types that force production, not just recognition.