# Micro-Expressions: A Comprehensive Guide

> **Author's Note:** This guide is the dedicated deep-dive on **micro-expressions** — the very brief, involuntary facial expressions (on the order of 1/25th to 1/5th of a second) that can leak a concealed emotion — covering the Paul Ekman science, the Facial Action Coding System (FACS), the seven universal emotions, the research evidence, the training tools, the applications (security, law enforcement, clinical, business), and — crucially — the honest limitations. It is written for solution architects, managers, and business professionals who negotiate, interview, present, and lead, and who want the *working* model: what micro-expressions actually are, how they are measured, what the evidence supports, and what it does **not** support. Where a topic has a dedicated sibling guide, this guide cross-references instead of duplicating: the *body-language and deception-detection context* (baselines, clusters of cues, the PEACE model, NLP eye patterns) lives in [body_language_lie_detection_guide.md](body_language_lie_detection_guide.md), the *conversation, feedback, and difficult-conversation skills* live in [communication_stakeholder_management_skills_guide.md](communication_stakeholder_management_skills_guide.md) and [team_lead_methodologies_guide.md](team_lead_methodologies_guide.md), the *negotiation and vendor due-diligence context* lives in [vendor_management_guide.md](vendor_management_guide.md), and the *AI/automated facial-recognition technology* lives in the AI guides under [../technology/ai_llm/](../technology/ai_llm/).
> *Part of the Management & Leadership Series | Author: Jack Liu Shurui — Solution Architect, Cymbal Bank*
> *Related guides in this series: [body_language_lie_detection_guide.md](body_language_lie_detection_guide.md) · [communication_stakeholder_management_skills_guide.md](communication_stakeholder_management_skills_guide.md) · [team_lead_methodologies_guide.md](team_lead_methodologies_guide.md) · [vendor_management_guide.md](vendor_management_guide.md) · [strategic_management_guide.md](strategic_management_guide.md) · [360_management_guide.md](360_management_guide.md) · [3d_managerial_effectiveness_guide.md](3d_managerial_effectiveness_guide.md) · [the_managers_path_research.md](the_managers_path_research.md)*

---

## How to Use This Guide

This guide is a reference manual and a field kit. **For a quick orientation**, read Section 1 (what micro-expressions are, and what they are not) and Section 9 (the honest verdict — including the single most important caveat: **micro-expressions reveal emotion, not truth**). **For the science**, Sections 2 and 3 cover the Facial Action Coding System and the seven universal emotions. **For the evidence**, Section 4 is the reality check — what research actually shows about recognition accuracy, training gains, and the failed security program that over-sold the science. **For building the skill**, Sections 5 and 7 cover training tools (METT/SETT, FACS certification) and practical technique. **For where it applies in your working life**, Section 6 covers security, law enforcement, clinical, and business applications. **To see it assembled**, Section 8 walks a single negotiation moment end-to-end — from a 150-millisecond flash to a probe question and an honest "one cue, one hypothesis" conclusion. The glossary at the end defines every term in one place.

Three reading notes. First, **this guide is deliberately sceptical**: the popular image of micro-expressions (Lie to Me, the TSA's behavior detection) outruns the evidence, and a large part of this guide is about what the research does *not* support — because an over-confident reader who "concludes from one flash" is worse off than a reader who knows the limits. Second, **micro-expressions are one channel**: the companion guide [body_language_lie_detection_guide.md](body_language_lie_detection_guide.md) covers the full body-language picture (baselines, cue clusters, posture, eye patterns) of which the face is one — arguably the most legible — part. Third, **where a claim is contested, unverifiable, or a simplification, it is flagged honestly** rather than glossed over; the "Verify" markers in Sections 1–4 show what is well-established science versus what is approximate or disputed.

---

## Table of Contents

1. [Micro-Expression Foundations](#1-micro-expression-foundations)
2. [The FACS System — Facial Action Coding System](#2-the-facs-system--facial-action-coding-system)
3. [The Seven Universal Emotions](#3-the-seven-universal-emotions)
4. [The Research Evidence — and Its Limits](#4-the-research-evidence--and-its-limits)
5. [Training Tools — METT, SETT, and FACS Certification](#5-training-tools--mett-sett-and-facs-certification)
6. [Applications — Security, Law Enforcement, Clinical, Business, Technology](#6-applications--security-law-enforcement-clinical-business-technology)
7. [Practical Guidance — How to Read Micro-Expressions](#7-practical-guidance--how-to-read-micro-expressions)
8. [Worked Example — A Contempt Flash in a Vendor Negotiation](#8-worked-example--a-contempt-flash-in-a-vendor-negotiation)
9. [Summary and the Honest Verdict](#9-summary-and-the-honest-verdict)
10. [Glossary](#10-glossary)
11. [References and Further Reading](#11-references-and-further-reading)

---

## 1. Micro-Expression Foundations

### 1.1 What a micro-expression is

A **micro-expression** is a very brief, involuntary facial expression — typically lasting between **1/25th of a second (about 40 milliseconds)** and **1/5th of a second (about 200 milliseconds)** — that appears on the face when a person experiences an emotion they are trying to conceal, and it "leaks" through despite the attempt at control. The defining properties, in order:

- **Brief.** The flash is too fast for the person to be fully aware of having shown it, and too fast for most observers to consciously register it. Durations under 1/5 second are the standard working definition (Ekman's own accounts use "less than one-fifth of a second"; the 1/25-to-1/5 range is the figure most widely quoted). *Verify: the exact duration bounds are Ekman's descriptive claim from clinical and laboratory observation, not a precisely measured physical constant — treat "1/25 to 1/5 second" as the widely cited ballpark, with some sources stretching to 1/2 second for the upper edge of "brief."*
- **Involuntary.** Unlike a deliberate smile or a staged frown, a micro-expression is generated by the emotion system itself. It is the face's automatic response to an emotional stimulus, and it escapes the person's management of their own face — which is precisely why it is interesting.
- **Full.** A micro-expression is not a partial expression; it is the *complete* expression of an emotion (all the muscle movements of, say, full anger) shown very briefly. This is what distinguishes it from a *subtle* expression (Section 1.3).
- **A leakage.** Its value — and its danger — is that it is a **leak**: it reveals the emotion the person is working to hide. The emotion is real; what follows from it (deception, hostility, agreement) is not automatic (Section 1.5).

A compact definition to carry into meetings: **a micro-expression is the full facial expression of a concealed emotion, shown for a fraction of a second, against the person's will.**

### 1.2 History — Ekman & Friesen (1969) and the birth of the concept

The micro-expression concept originates with psychologist **Paul Ekman** and his long-time collaborator **Wallace V. Friesen**. In their 1969 paper **"Nonverbal Leakage and Clues to Deception"** (*Psychiatry*, 32(1), 88–106), they introduced the idea of **nonverbal leakage** — the notion that when people try to conceal what they are feeling, their nonverbal behavior (face, body, voice) "leaks" the concealed information in ways that are hard to control — and described the micro-expression as one of the most important leakage channels. *Verified: the paper is real — Ekman, P., & Friesen, W. V. (1969), Psychiatry, Vol. 32, No. 1, pp. 88–106, DOI 10.1080/00332747.1969.11023575 (PubMed ID 5779090).*

The origin story, as Ekman tells it: while reviewing films of a psychiatric patient who had concealed her intent to commit suicide, Ekman and Friesen found that when they slowed the film to a frame-by-frame pace, they could see expressions that were invisible at normal speed — the patient showed brief flashes of distress that contradicted her cheerful presentation. (This anecdote is Ekman's own account; it is widely repeated but the details vary across tellings.) The 1969 paper established the vocabulary — *leakage*, *micro* vs *macro* expressions — that still structures the field.

A crucial historical note: the 1969 paper's title puts "deception" front and center, and the popular imagination has married micro-expressions to lie detection ever since. The paper was about **clues to deception** — but the modern, evidence-based reading is much more careful (Section 1.5 and Section 4): a micro-expression tells you an *emotion* leaked, not that a *lie* was told.

The 1969 paper also did two other things that still structure the field. First, it laid out a **typology of nonverbal leakage channels** — the face, the body, the voice, and speech style — and argued that each channel has a different "leakage potential" depending on how well the sender monitors and controls it; the face, ironically, is both the most expressive and the most managed channel, which is why its *brief* failures (micros) matter so much. Second, it introduced the **micro/macro distinction** in explicit terms: the "micro" expression as the very brief, complete, hard-to-control expression, versus the "macro" expression as the ordinary, longer, more manageable one. Everything in Sections 2–9 is an elaboration of that 1969 skeleton — which is why the field still dates its origin to that single paper.

### 1.3 Micro vs macro vs subtle expressions

Ekman's framework distinguishes three types of facial expression, and conflating them is the most common error in casual body-language reading:

| Type | Duration | Intensity | What it is | Example |
|---|---|---|---|---|
| **Micro-expression** | < 1/5 s (often 1/25–1/5 s) | Full intensity | The complete expression of an emotion, flashed briefly; typically a leakage of a concealed emotion | A full anger face — brows down, eyes hard, lips tight — for 120 ms, immediately replaced by a smile |
| **Macro-expression** | ~1/2 s to 4 s | Full intensity | The normal, visible expression — what people show when they are not (or not successfully) hiding an emotion | A held smile of pleasure at good news; a sustained frown |
| **Subtle expression** | Variable (can be long) | Low intensity / partial | Only part of the expression appears (e.g., only the eyes, or only a slight lip movement); the sign of a *suppressed* or *dampened* emotion, or of the very beginning of an emotion | The corners of the mouth barely tighten while the rest of the face stays neutral — a hint of disgust with no nose wrinkle |

The practical implication: when you see a flash of emotion, ask *which type* you saw. A **micro** expression is a full emotion leaking (high information, very brief). A **subtle** expression is a partial or low-intensity emotion showing (lower intensity, can last longer). A **macro** expression is the person's managed or unmanaged face (the most common thing you see, and the least diagnostic on its own).

### 1.4 Leakage and "hot spots"

**Leakage** is Ekman's umbrella term: the involuntary transmission of concealed information through nonverbal channels — face, body, voice, speech style — that the person is not monitoring or cannot fully control. Micro-expressions are the *facial* form of leakage; the companion guide [body_language_lie_detection_guide.md](body_language_lie_detection_guide.md) covers the body and voice forms.

**Hot spots** (Ekman's term) are the *moments* where leakage is most likely: the points in a conversation where the emotional stakes suddenly rise — a question about money, a mention of a rival, a rejected counter-offer, an accusation, a name. At a hot spot, the concealed emotion is most likely to surface as a micro-expression, because the emotion is freshly triggered and the person's management resources are focused on their words. In practice: **do not scan the face continuously; watch the face *at the hot spots*.** This is the single most useful operational translation of the science (Section 7.4).

### 1.5 The critical distinction — emotion ≠ deception

The most important idea in this guide, and the one most often lost in popular treatments:

> **A micro-expression shows you the *emotion* a person is feeling. It does not tell you whether they are lying, and it does not tell you the *reason* for the emotion.**

A contempt micro-expression at your price proposal does not mean "the vendor is lying about the price." It means "the vendor felt contempt at that moment" — which could mean the price is a non-starter, or that they despise your negotiation style, or that they were contemptuous of the *process*, or any of a dozen things. The emotion is a **lead to investigate, not a verdict to render**. This is the honest framing that runs through this entire guide, and it matches the companion guide's treatment: in [body_language_lie_detection_guide.md](body_language_lie_detection_guide.md) the same distinction appears as "cues are evidence of emotion, not of deception." A micro-expression is therefore best treated as: *a signal that an emotion was present, which generates a hypothesis worth probing with a question — nothing more* (see the worked example in Section 8).

### 1.6 A note on terms — micro-expression, microexpression, and the vocabulary

The field's terminology has settled into a few stable forms worth knowing, because search and reading will throw all of them at you: **"micro-expression"** (the standard hyphenated form, used throughout this guide), **"microexpression"** (the unhyphenated variant, common in the academic literature and in compound names like JACBART-style tests), and **"micro expression"** (the open form, most common in popular writing). All three refer to the same phenomenon; the hyphenated form is used here for consistency. Related terms to recognize: **"micro-facial expression"** (an occasional synonym emphasizing the face), **"brief affect"** (the academic register — hence "Brief Affect Recognition Test"), and **"micro-momentary expression"** (an early synonym from the 1960s literature). When reading critically, be alert to the *marketing* variant: **"micro-expression analysis"** sold as a service often implies the very claims — lie detection, intent detection, personality profiling — that the evidence does not support (Sections 4.4 and 6.4); the honest vocabulary of this guide is *observation, hypothesis, and probe*.

---

## 2. The FACS System — Facial Action Coding System

### 2.1 What FACS is

The **Facial Action Coding System (FACS)** is the standard, comprehensive system for describing facial movement — the "periodic table" of the face. Developed by **Ekman and Friesen**, the manual was **first published in 1978** and **most recently revised in 2002** (Ekman, Friesen, & Hager, *FACS Manual*, 2002). *Verified: 1978 original and 2002 revision are the universally cited dates (the Paul Ekman Group's own FACS page confirms "first published in 1978... most recently revised in 2002").*

FACS works by decomposing any visible facial movement into its constituent **Action Units (AUs)** — anatomically grounded units of muscle movement — rather than by naming emotions. The genius of the design is that it is **emotion-free**: a FACS coder does not decide "this face is sad"; they code "AU1 + AU4 + AU15 are present," and the emotion label (if any) is a separate interpretive step. This separation is what made FACS the research gold standard: it gives objective, reliable, language-independent measurement of facial behavior.

### 2.2 The Action Units (AUs)

The 1978 FACS defined **44 Action Units**; the 2002 revision expanded the catalogue (the full system today describes on the order of 44–46 AUs plus numerous "miscellaneous" movements, head/eye positions, and intensity variants). *Verify: "44" is the classic count; the 2002 edition added units (e.g., AU46 — the wink) and formalized descriptors, so "44–46" is the honest range, and different sources quote different totals.*

The key AUs a reader of this guide will actually encounter (all verified against the standard FACS numbering):

| AU | Name | What it looks like (muscle) |
|---|---|---|
| **AU1** | Inner brow raiser | Inner corners of the brows lift (frontalis, medial portion) — the sad/surprised brow |
| **AU2** | Outer brow raiser | Outer brows lift (frontalis, lateral portion) — with AU1, the full raised brow of surprise/fear |
| **AU4** | Brow lowerer | Brows pulled down and together (corrugator, depressor supercilii) — anger, concentration, distress |
| **AU5** | Upper lid raiser | Eyes widen (levator palpebrae) — surprise, fear, alertness |
| **AU6** | Cheek raiser | Cheeks lift, "crow's feet" appear at the outer eyes (orbicularis oculi, pars orbitalis) — the hallmark of the genuine smile |
| **AU7** | Lid tightener | Eyelids tighten/narrow (orbicularis oculi, pars palpebralis) — anger, suspicion, squinting |
| **AU9** | Nose wrinkler | Nose wrinkles, nostrils flare (levator labii superioris alaeque nasi) — disgust |
| **AU10** | Upper lip raiser | Upper lip lifts (levator labii superioris) — disgust, contempt-adjacent, sneering |
| **AU12** | Lip corner puller | Corners of the mouth pull up and out (zygomatic major) — the smile muscle |
| **AU14** | Dimpler | Corners tighten inward, dimples may appear (buccinator) — the "tight smile"; core of contempt |
| **AU15** | Lip corner depressor | Corners pull down (depressor anguli oris) — sadness, displeasure |
| **AU17** | Chin raiser | Chin/lower lip pushes up (mentalis) — sadness, pouting, stubbornness |
| **AU20** | Lip stretcher | Lips pull horizontally (risorius) — fear |
| **AU23** | Lip tightener | Lips press together and thin (orbicularis oris) — anger, controlled tension |
| **AU24** | Lip pressor | Lips press firmly together (orbicularis oris) — anger, determination |
| **AU25–27** | Mouth opening | AU25 lips part; AU26 jaw drops; AU27 mouth stretches — surprise, fear (jaw open) |
| **AU43** | Eye closure | Eyes close (relaxation of levator; not a blink) — deliberate closure, often pain or overwhelm |
| **AU45** | Blink | The blink (included in the system for completeness) |

*Verify: the AU numbers above are the standard FACS numbering and match the published FACS manual and the Paul Ekman Group's AU list; AU46 (wink) exists in the 2002 revision, which is why "44–46" appears in the literature. AU43 is technically "eye closure" and AU45 "blink" — both are in the system, though AU45 is often excluded from emotion research because blinks are frequent and mostly non-emotional.*

### 2.3 FACS scoring and emotion prototypes

FACS scoring is the act of watching (usually frame-by-frame or slow-motion video) and recording which AUs occur, when, at what intensity (rated A–E), and on which side of the face (unilateral vs bilateral is highly diagnostic — see contempt, Section 3.2). A single minute of video can take an experienced coder around an hour to score; reliability is measured as agreement with expert criterion codes, with **0.70 agreement** the standard certification threshold (Section 2.4). *Verify: the ~1 hour of scoring per minute of video and the 0.70 agreement threshold are figures cited in FACS training materials and by certified-coder trainers; treat them as the conventional benchmarks.*

Emotion "prototypes" — the AU combinations that research associates with each of the seven universal emotions — are derived from FACS-coded expressions of the emotions, and they are the bridge between the mechanical AU system and the interpretive emotion system:

| Emotion | Core AU combination (prototype) |
|---|---|
| Happiness | **AU6 + AU12** — the **Duchenne smile** (see below) |
| Sadness | AU1 + AU4 + AU15 (+ AU17) — inner brows up, brows down, lip corners down |
| Anger | AU4 + AU5 + AU7 (+ AU23/AU24) — brows down, eyes wide, lids tight, lips tight |
| Fear | AU1 + AU2 + AU4 + AU5 + AU20 (+ AU25/26) — brows raised and drawn together, eyes wide, lips stretched |
| Disgust | AU9 (+ AU10, AU15/16) — nose wrinkle, upper lip raised |
| Surprise | AU1 + AU2 + AU5 (+ AU26/27) — brows up, eyes wide, jaw dropped |
| Contempt | **AU12 + AU14** — or unilateral AU14 — the sneer (Section 3.2) |

*Verify: these prototype combinations are the standard ones taught by Ekman's own materials and reproduced across FACS textbooks; there is variation across sources in the *exact* AU list per emotion (e.g., whether AU7 belongs in anger), and the prototypes are descriptive averages, not a law — individuals vary. The happiness = AU6 + AU12 identity is the most robust and most replicated claim in the whole system (the Duchenne smile, below).*

### 2.4 FACS training and certification

Becoming a **FACS coder** is a serious commitment. The original manual was designed to be self-instructional, and the conventional figure is that **at least 100 hours of study and practice** are required to reach minimal competency (certified-coder trainers and FACS training sites consistently cite "100+ hours"). Certification itself is achieved by passing the **FACS final test** — coding a set of video segments and achieving at least **0.70 agreement** with expert criterion codes. The Paul Ekman Group runs FACS certification workshops; researchers who need published-grade facial coding (for academic studies, clinical trials with facial endpoints, or product research) are expected to be certified or to work under certified coders. *Verify: the 100+ hour figure and the 0.70-agreement final test are the figures cited by FACS training providers and certified coders (e.g., the Rosenberg FACS training page and the Paul Ekman Group); they are conventional benchmarks rather than a single canonical regulation.*

For the business reader, the takeaway is not "get certified" (a large investment for a manager) but **what certification implies about the difficulty of the task**: reliable facial coding is a 100-hour skill. Anyone who claims to "read micro-expressions fluently" without that kind of investment is over-claiming (Section 4 and Section 7.5).

### 2.5 Automated FACS — computer vision and facial coding software

Since the late 2000s, computer vision has partially automated AU detection. The leading tools:

- **Affectiva** (spun out of MIT Media Lab; emotion/engagement metrics from webcam video, widely used in ad testing and automotive driver monitoring),
- **iMotions** (a biosensor/eye-tracking platform with an integrated facial-expression analysis module),
- **Noldus FaceReader** (one of the oldest automated FACS-style systems; classifies the seven basic emotions plus valence/arousal),
- **OpenFace** (open-source AU detection and facial landmark tracking — the free research standard).

These systems detect AUs (and by extension emotion categories) at rates far beyond human perception — hundreds of frames per second — but their accuracy degrades with head pose, lighting, occlusion, and — critically for this topic — **they are tuned for macro expressions at normal intensity; micro-expressions (40–200 ms) remain much harder to detect reliably**, and their emotion *labels* inherit all the interpretive limits discussed throughout this guide. Automated facial coding is a real and maturing technology (see [../technology/ai_llm/](../technology/ai_llm/) for the AI side), but "the software can see it" does not mean "the software knows what it means." Section 6.5 covers the regulatory and ethical dimension (EU AI Act).

### 2.6 FACS applications beyond emotion research

FACS is used well beyond psychology:

- **Research** — the gold standard for measuring facial behavior in emotion studies, psychopathology research, pain assessment, and parent–infant interaction studies.
- **Facial animation (CGI/VFX)** — FACS-based animation is the industry standard for realistic digital faces: character rigs are built on AU equivalents so animators move the face in anatomically correct units (the same reason FACS is used in facial-avatar systems and, increasingly, in real-time digital humans).
- **Clinical trials and product testing** — facial endpoints (e.g., "does this analgesic reduce the pain expression?") and consumer-emotion measurement in advertising and UX research.
- **Security and deception research** — the (contested) foundation of behavior-detection programs (Section 6.1).

### 2.7 Reading FACS in practice — intensity, laterality, and the limits of AU coding

Three coding concepts matter even for non-coders, because they map directly onto what you can and cannot conclude from a face:

- **Intensity (A–E).** FACS rates each AU on a five-point intensity scale (A = trace, E = maximum). A trace AU4 (barely-there brow lowering) is a *subtle* expression; a maximum AU4+AU5+AU23 is full anger. Intensity is the formalization of the micro/macro/subtle distinction (Section 1.3): the same AU at different intensities means different things conversationally — a trace is often a suppressed or incipient emotion, a full blast is an unmanaged one.
- **Laterality (unilateral vs bilateral).** Many AUs can occur on one side of the face or both. One-sided movements are a red flag for *deliberate* production in the emotion literature — genuine emotional expressions are typically symmetrical — and the one major exception is contempt, which is *normally* unilateral (Section 3.2). So laterality is doubly diagnostic: asymmetry suggests either deliberation or contempt, and telling those apart requires context.
- **The limits of AU coding.** FACS describes *what moved*, not *why*. The same AU4 (brow lowering) appears in anger, concentration, confusion, pain, and squinting against light — the AU is unambiguous, the *meaning* is not. This is the formal version of the guide's core caveat: even perfect FACS measurement produces a movement description, and the emotion inference is a separate, weaker step. Automated systems (Section 2.5) inherit exactly this structure — they measure movements well and infer meaning poorly.

---

## 3. The Seven Universal Emotions

### 3.1 The seven

Ekman's cross-cultural research identified a set of **basic emotions with universally recognized facial expressions**. The classic list has **seven**: **anger, fear, sadness, disgust, surprise, contempt, and happiness**. *Verify: Ekman's original cross-cultural work (1960s–70s) established six — anger, fear, sadness, disgust, surprise, happiness; **contempt was added later** (Ekman's cross-cultural work on contempt appeared in the 1990s, e.g., Ekman & Heider 1988 and Ekman's later writings — most sources date the "seven" formulation to the 1990s). The seven-item list above is the one used by the Paul Ekman Group and by virtually all training tools (METT/SETT).*

The important caveats that come with "universal":

- **Universality means *recognition* and *production* of the expression are cross-culturally shared** — Ekman showed that members of a visually isolated culture (the Fore people of Papua New Guinea) could both produce and correctly label the same expressions Westerners used (Section 3.4). It does **not** mean everyone *displays* them the same way — **display rules** (Section 3.3) govern when an expression is shown, to whom, and in what form.
- **The list is not exhaustive** — these are the seven with the strongest cross-cultural recognition evidence; Ekman himself has discussed additional candidate emotions (embarrassment, amusement, pride, etc.) and the "basic emotions" framework is a live scientific debate (Section 4.4).
- **The expressions are averages, not laws** — prototypes describe the typical full expression; real faces mix, mask, and miniaturize them.

### 3.2 The facial signature of each emotion

These are the FACS prototypes (Section 2.3) rendered in plain language — what to look for when you deliberately watch a face:

- **Anger** — *brows down and together (AU4), eyes wide (AU5) with tightened lids (AU7), lips pressed or tightened (AU23/AU24)*; the face narrows and hardens. The classic anger face is "brows down, eyes hard, mouth compressed."
- **Fear** — *brows raised and drawn together (AU1+2+4 — the distinctive "fear brow" that combines raised and pulled), eyes very wide (AU5), lips stretched horizontally (AU20), mouth may be open (AU25/26)*; the face opens and pulls back.
- **Sadness** — *inner brows raised and pulled up (AU1), brows down (AU4), lip corners pulled down (AU15), chin raised (AU17)*; the face droops, especially the upper eyelids and the corners of the mouth.
- **Disgust** — *nose wrinkled (AU9), upper lip raised (AU10), possibly lip corners down (AU15) or lower lip pushed up (AU16)*; the face "turns away" from the offending stimulus, centered on the nose and mouth.
- **Surprise** — *brows raised (AU1+2), eyes wide (AU5), jaw dropped (AU26/27)*; the face opens upward and outward. Surprise is the shortest-lived of the macro emotions and the easiest to fake (a raised brow and open mouth are cheap to produce deliberately).
- **Contempt** — *one side of the mouth tightened and lifted — the unilateral lip corner (AU12 on one side, or AU14 — the dimpler) — the sneer*; the face is asymmetrical, and this asymmetry is the diagnostic feature. *Verify: contempt is commonly described as AU12+14 (combined) or as unilateral AU14; Ekman's materials describe it as a unilateral tightening of the lip corner. It is the only universal emotion whose expression is typically asymmetrical — which is precisely why the AU14 dimpler and laterality matter in FACS coding.*
- **Happiness** — *the **Duchenne smile**: lip corners up (AU12) **plus** cheek raise and crow's feet at the eyes (AU6)*; the genuine smile engages the eye muscle (orbicularis oculi) that cannot be contracted voluntarily by most people — which is why "smiling eyes" are the marker of genuine enjoyment, and why a smile without AU6 (the "non-Duchenne" or "social/polite" smile) is the mask of choice in business. *Verify: the Duchenne distinction — named after the 19th-century French anatomist Guillaume Duchenne, whose work Darwin cited — is one of the most replicated findings in facial-expression research; AU6 (cheek raiser / crow's feet) is the discriminator. Note that the Duchenne smile is a marker of *felt positive emotion* in most studies, not of truthfulness.*

### 3.3 Display rules — why the same emotion looks different across cultures

**Display rules** (a term Ekman & Friesen introduced in the 1969 paper) are the culturally learned norms that govern the *management* of emotional expression: which emotions may be shown, to whom, when, and in what intensity. They are the reason universality of *recognition* coexists with huge *display* variation:

- **Neutralization/suppression** — hiding the emotion behind a neutral face (the Japanese students in Ekman's early studies who, watching stressful film clips, showed the same facial responses as Americans when alone but *neutralized* them when an experimenter was present — the classic demonstration of display rules).
- **Masking** — replacing the true expression with a different one (the smile that covers sadness or anger).
- **Intensification / de-intensification** — amplifying or dampening the expression (the "customer-service smile" turned up; the Western stiff-upper-lip turned down).
- **Qualifying** — adding a second expression on top (the "smile with a hint of worry" of someone receiving an award they didn't expect).

The operational consequence: **before interpreting any expression — micro or macro — you need to know the person's baseline and their display-rule context** (see [body_language_lie_detection_guide.md](body_language_lie_detection_guide.md) on baselines). A suppressed smile in a Japanese business meeting and a suppressed smile in an American one are not the same event, even if the muscles are identical.

### 3.4 The masking story — "macro to micro"

The classic micro-expression scenario is **masking**: the person manages their face into a socially appropriate macro expression (the smile at the client's joke, the calm nod at the rejected proposal) while a different emotion — sadness, anger, contempt — leaks through as a micro-expression underneath or around it. This is what Ekman called the "macro-to-micro" sequence: macro mask first, micro leak in the transition (or vice versa — the micro flash before the mask snaps into place). The leak often appears at the *edges* of the mask: the instant before the smile, the instant after it, or in the muscles the mask does not cover (the eyes when the mouth smiles). In practice, this is why the moment *between* expressions is more informative than the expressions themselves.

### 3.5 Recognition — cross-cultural evidence and emotional intelligence

**Universal recognition.** Ekman's studies from the 1960s–70s remain the foundational evidence: photographs of posed expressions of the six (later seven) emotions were shown to members of cultures with no exposure to Western media — most famously the **Fore people of Papua New Guinea** — who matched the expressions to emotion stories at rates far above chance, and whose own posed expressions were in turn correctly labeled by Western observers. *Verified: the Fore studies (Ekman, Sorenson & Friesen 1969; Ekman & Friesen 1971; Ekman's book "The Face of Man", 1980, documents the New Guinea fieldwork) are the canonical universality evidence. Later replications and meta-analyses (including David Matsumoto's cross-cultural work across dozens of countries) broadly support universal recognition of the basic expressions, though with cultural differences in *intensity judgments* and *display*, and with the expected critiques (see Section 4.4).*

**Matsumoto's contribution.** Psychologist **David Matsumoto** (San Francisco State University) extended the universality program cross-culturally and built the key measurement tool for micro-expression research: the **JACBART** (Japanese and Caucasian Brief Affect Recognition Test), which presents emotions at micro durations and is the standard instrument for measuring micro-expression recognition ability — and for measuring the training gains discussed in Section 5. *Verify: JACBART (Matsumoto et al., 2000) presents brief (1/25 s) flashes of Japanese and Caucasian faces; it is the standard research instrument in this literature.*

**Emotional intelligence.** Emotion recognition — accurately reading others' faces — is a core component of emotional intelligence (the "perceiving emotion" branch of the Mayer–Salovey–Caruso model, and a central plank of the Goleman popularization). The management-series guides treat EI in depth ([communication_stakeholder_management_skills_guide.md](communication_stakeholder_management_skills_guide.md), [team_lead_methodologies_guide.md](team_lead_methodologies_guide.md)); the honest research position is that *basic* emotion recognition is a real, measurable, trainable skill — and that micro-expression recognition specifically is a *harder, narrower* skill than the EI literature usually implies (Section 4).

### 3.6 Reading the whole face — eyes vs mouth

Two practical reading heuristics that sit on top of the seven prototypes:

- **The upper face is harder to fake than the lower face.** The mouth is the most controllable region — anyone can produce a smile on demand. The eyes and brows are harder: AU6 (the cheek raise/crow's feet of the genuine smile) and AU1/AU4 (the inner-brow patterns of sadness and distress) involve muscles that most people cannot contract cleanly and voluntarily. This is the anatomical basis of the old advice "trust the eyes, not the mouth" — and of the Duchenne distinction. When words and the lower face say one thing and the upper face says another, weight the upper face.
- **Confusable pairs.** In real time, and especially in micros, several emotion pairs are easy to mix up: **fear vs surprise** (both have raised brows and wide eyes — the discriminator is the brow *shape*: fear draws the brows together, AU1+2+4, while surprise only raises them, AU1+2, and the mouth: fear stretches (AU20), surprise drops open (AU26/27)); **anger vs disgust** (both can involve tightened lips — anger is brows-down + hard eyes, disgust is nose-wrinkle + upper-lip raise); **sadness vs fear** (both can show inner-brow raise — sadness has drooping lip corners (AU15), fear has wide eyes and stretched lips). When you cannot decide between two prototypes, say so — "fear or surprise" is a legitimate, useful observation; forcing it to one label is where errors enter.

### 3.7 The frequency question — how often do micros actually occur?

A question every practitioner eventually asks: how often do real micro-expressions appear in ordinary conversation? The honest answer is that **the natural-occurrence base rate is not well established** — the field's measurements come mostly from laboratory elicitation (staged lies, emotion-inducing films, posed stimuli), and studies that count micros in natural conversation report a wide range depending on definition and setting. What is reasonably supported: micros are *more* likely at hot spots (Section 1.4), *more* likely when the emotion is freshly triggered and the suppression attempt is recent, and *less* likely in highly practiced, scripted interactions (a seasoned negotiator delivering a rehearsed line is a poor micro source). Two practical consequences: (a) do not expect to see a dramatic full micro every meeting — the subtle expression (Section 1.3) is the more common real-world leakage form; (b) treat a *lack* of micros as meaningful only as "no leakage observed," never as "no emotion present" — skilled suppressors leak less, and some people's faces simply move less (Section 4.4, individual differences).

---

## 4. The Research Evidence — and Its Limits

### 4.1 What the research actually shows — recognition accuracy

The honest headline of the micro-expression literature: **untrained observers are bad at seeing micro-expressions, and training makes them measurably better — but the "bad" baseline and the "better" endpoint are both far below the popular image of the skill.**

- **Untrained recognition is low.** In studies presenting expressions at micro durations (1/25 s, the classic JACBART presentation), untrained observers' recognition accuracy is routinely reported at **roughly chance level to modestly above chance** — commonly cited as in the region of **30–50%** depending on the emotion and the test, with some individual emotions (e.g., contempt, disgust) recognized well below others (e.g., happiness). *Verify: exact figures vary by study and instrument; the consistent finding across Matsumoto's JACBART studies and replications is that untrained recognition of 1/25 s flashes is poor — commonly at or only modestly above chance — and the "~50%" figure frequently quoted in the popular literature is a rough average of such results, not a single canonical number. The task instruction's "~50%" is consistent with the widely cited ballpark; treat it as approximate.*
- **Training improves recognition.** The most replicated applied finding is that **brief training (typically the METT, Section 5) produces large, statistically reliable gains** in micro-expression recognition — commonly reported as moving people from poor/chance-level performance to roughly 60–80% accuracy on the trained stimuli, with gains persisting at least over weeks-to-months in most studies. *Verify: METT training studies (e.g., Ekman 2002/2003 program evaluations; Matsumoto and Hwang's studies of METT-style training) consistently show significant pre-to-post gains; the precise magnitude varies by study. A fair summary: "training roughly doubles recognition accuracy from a poor baseline" — the popular "from ~50% to ~80%" shorthand is a reasonable, if loose, rendering.*
- **Gains do not automatically transfer to real life.** Recognition of *laboratory* stimuli (posed, clean, well-lit, single emotion) is not the same as spotting a flash in a moving, multi-channel, emotionally loaded conversation. Training studies measure the former. The lab-to-field gap is one of the deepest problems in this literature (Section 4.4).

### 4.2 Key studies

- **Ekman and Friesen (1969)** — the foundational leakage paper (Section 1.2); established micro vs macro and the leakage concept. *Verified.*
- **Ekman, Sorenson and Friesen (1969) and Ekman and Friesen (1971)** — the cross-cultural universality studies including the Fore of Papua New Guinea (Section 3.4). *Verified.*
- **Ekman and Friesen (1978)** — the FACS manual. *Verified.*
- **Matsumoto et al. (2000)** — JACBART: the standard instrument for measuring micro-expression recognition; also documented the poor untrained baseline. *Verified as the standard instrument; specific accuracy figures vary by study.*
- **Matsumoto and Hwang (2011)** — "Evidence for training the ability to read microexpressions of emotion" (Motivation and Emotion): showed significant gains from a training intervention, including transfer to improved detection in a broader test. *Verified: this is the most-cited modern training-efficacy study in the micro-expression literature.*
- **Matsumoto, Frank and Hwang (2013)** — "Reading people: introduction to the special issue on nonverbal communication" and related work on individual differences (personality correlates of recognition ability; e.g., younger and more open individuals score higher — but individual differences did *not* predict who improved most from training). *Verified: the individual-differences finding (age, openness predicting baseline ability but not training gain) comes from the published studies in this line (e.g., Matsumoto and Hwang's background-factors work).*
- **Frank, Maccario and Govindaraju (2009)** — "Behavior and security" and related work showing that **behavior-based deception detection, including micro-expression reading, fails to reach the accuracy needed for security screening** — a key skeptical data point (Section 4.3). *Verified: Frank et al.'s analyses of behavior-based screening accuracy (finding accuracy rates far below what security screening requires, often near chance) are a standard citation in the skeptical literature.*

### 4.3 The cautionary tale — TSA SPOT

The largest real-world deployment of behavior-detection (including micro-expression concepts) is the US Transportation Security Administration's **SPOT** program — **Screening of Passengers by Observation Techniques** — which ran from **2007** and grew to roughly **3,000 Behavior Detection Officers at more than 160 airports**. Its premise: trained officers would spot behavioral indicators (including micro-expressions, "stress indicators," and other nonverbal cues) of hostile intent, and refer suspicious passengers for secondary screening.

The outcome, as documented by the US Government Accountability Office (GAO):

- **GAO (2010, GAO-10-763)** and **GAO (2013, GAO-14-159)** both concluded that **TSA had not demonstrated SPOT's effectiveness** — no validated evidence that the behavioral indicators actually identified terrorists (vs. generating large numbers of referrals that never produced a terrorism arrest), no scientific validation of the indicators, and no measurement of the false-positive burden on passengers.
- Academic analyses (e.g., Frank et al. 2009; Weinberger 2010 in *Nature*) reinforced the point: the base-rate problem makes behavior-based screening mathematically hopeless — when the event being sought is vanishingly rare, even a good detector produces almost exclusively false positives (Section 4.4).
- The program was **renamed Behavior Detection and Analysis (BDA) in 2016** and continued in reduced form; it was **never validated by the evidence**, and critics — including the American Civil Liberties Union, which documented widespread complaints of ethnic profiling — kept it under sustained attack. *Verify: program dates (launched 2007; ~3,000 officers/160+ airports by 2013 per GAO-14-159; renamed BDA in 2016), the GAO's "effectiveness not demonstrated" conclusions, and the FY2015 funding of ~$189.4M are all documented. The current operational status is fluid and not independently verified here — treat "scaled back, unvalidated, and controversial" as the fair description.*

The SPOT story is the single most important real-world lesson of this guide: **an unvalidated application of facial/behavioral reading, deployed at scale, produced exactly what the math predicted — expense, harassment, and no demonstrated security gain.** Whenever a vendor, tool, or program promises to "detect deception" or "flag hostile intent" from faces, SPOT is the reference cautionary tale.

### 4.4 The honest limitations — what the science does not support

The full list of limits, stated plainly:

1. **The lab-vs-field gap.** Nearly all recognition evidence comes from posed, isolated, well-lit stimuli presented in tests. Real conversations are moving, multi-channel, occluded, culturally embedded, and emotionally mixed. There is **no field study showing that trained micro-expression readers reliably outperform chance at detecting concealed emotions in live, high-stakes interactions.**
2. **The base-rate problem.** In any real screening or detection application, the target event (lying, hostile intent, fraud) is rare. With a rare base rate, even a detector with good sensitivity produces an overwhelming number of **false positives** — the SPOT outcome. This is arithmetic, not psychology.
3. **Individual differences.** Expressions are not identical across people: some people have naturally strong or weak expressions (the "poker face" spectrum), habitual expressions, and culture- and personality-driven display rules. Baselines differ; prototypes are averages. Misreading an individual's normal face as a leak is the most common practical error.
4. **Emotion ≠ deception (again, louder).** The evidence supports *"a brief full expression of an emotion occurred."* It does not support *"the person is lying,"* *"the person is guilty,"* or *"the person intends harm."* Ekman himself has repeatedly stressed that micro-expressions indicate emotion, and emotion can be concealed for innocent reasons (surprise at bad news, fear of the interviewer, contempt for the process).
5. **Micro-expressions are not a polygraph.** They carry none of the physiological inference of a polygraph — and the polygraph itself is scientifically contested. The honest framing: micro-expression reading is a *hypothesis generator* for conversation, not a *truth detector* for evidence.
6. **The critique of the universality program itself.** The "basic emotions" framework has serious scholarly critics (e.g., Lisa Feldman Barrett's constructed-emotion theory, which argues that emotion categories are cultural constructs and that facial movements map onto *valence/arousal* rather than discrete emotion categories; and critiques of the Fore studies' methodology). The universality claim is *well-supported but not settled* — a fair summary is "the seven expressions show strong cross-cultural recognition, with real cultural and individual variation, within an active scientific debate about what emotions fundamentally are."
7. **Publication and replication context.** The field is small; many key findings rest on a handful of labs (Ekman's, Matsumoto's); independent replication of the *training-transfer* and *field-utility* claims is thinner than the applied enthusiasm would suggest.

**The honest summary of the evidence:** micro-expressions are real (the phenomenon exists; FACS measures it reliably; recognition is cross-culturally shared and trainable in the lab), and practically useless as a stand-alone detection technology in the field — but genuinely useful as a *conversational signal* for people who understand its limits (Section 7).

### 4.5 What training actually buys you — and does not

Given the marketing around micro-expression courses, it is worth being precise about the evidence for training:

- **What training buys:** reliably better recognition of *brief, isolated, posed expressions* — the METT/SETT skill. Untrained people hover near chance on 1/25 s flashes; trained people reliably identify the emotion in the trained stimuli. That is a genuine, replicated, measurable gain, and it is the whole legitimate basis of the training industry.
- **What training does not buy:** (a) field accuracy — no study shows trained readers reliably detecting concealed emotions in live, high-stakes interaction; (b) deception detection — training improves *emotion labeling*, and emotion ≠ deception (Section 1.5); (c) immunity to context — trained readers still misread when they lack the person's baseline, the display-rule context, or the conversational facts; (d) durability — recognition skill decays without practice, so a one-off course is a one-off gain.
- **The meta-lesson:** the training literature is one of the cleanest demonstrations of the *illusion of explanatory depth* — a little training makes people feel much more skilled than they measurably are in the field. The professional response is not to skip training (it is worth doing) but to *keep score*: log your reads against outcomes and let your personal hit rate, not the certificate, be the calibration (Section 7.3).

---

## 5. Training Tools — METT, SETT, and FACS Certification

### 5.1 METT — the Micro Expression Training Tool

**METT** (the **Micro Expression Training Tool**) is Ekman's self-instructional training program, distributed through the **Paul Ekman Group**. Its design:

- A **pre-test** of micro-expression recognition (JACBART-style brief presentations);
- A training phase teaching the seven emotions' facial signatures — with video demonstrations and deliberate practice at increasing speeds;
- A **post-test** measuring the gain.

*Verified: METT is real and current (sold via paulekman.com's "Micro Expressions Training" tools page); it is the most-studied micro-expression training instrument; the pre/post structure is as described. The related **SETT — Subtle Expression Training Tool** — trains recognition of *subtle* expressions: low-intensity, partial expressions (Section 1.3), i.e., emotions that are suppressed or just beginning, which are even harder to see than micros. The Paul Ekman Group sells METT, SETT, and combined packages; an online course version exists. Verify note: exact current pricing and packaging change over time and are not verified here.*

The training-effectiveness evidence (Section 4.1) is built substantially on METT: pre/post studies consistently show significant gains, and the Matsumoto and Hwang (2011) study demonstrated both gains and some transfer to broader recognition tests. Two honest caveats: (a) gains are on *laboratory* stimuli — transfer to live conversation is unproven; (b) skills **fade without practice** — the popular guidance is that recognition ability decays within months if not maintained (a claim consistent with the training literature's general findings, though precise decay curves are not well established).

### 5.2 SETT — the Subtle Expression Training Tool

**SETT** addresses the *other* leakage channel: not the brief flash (micro) but the **low-intensity or partial** expression (subtle) — the faint tightening of the lips, the barely-there brow movement — which is arguably the more common real-world leakage form in controlled professional settings (people in business meetings usually suppress *hard*; full micros are less common than partial subtles). Training both (METT + SETT) is the standard recommended package from the Paul Ekman Group. For the business reader: **SETT's object — partial, low-intensity expressions — is closer to what you will actually see in a meeting than the dramatic 100 ms flash.**

### 5.3 Deliberate practice and the road to real skill

The evidence-based route to genuine (if limited) skill:

1. **Learn the FACS system** — at least the core AUs (Section 2.2) and the seven prototypes (Section 2.3), even without certification. This gives you a vocabulary for *what* you are seeing ("AU4 + AU7 — that's brow-lowering with lid tightening") instead of vague impressions.
2. **Train with METT/SETT** and repeat periodically — the pre/post structure gives you honest feedback, which is the essence of deliberate practice. (Unstructured "people watching" without feedback does not build the skill; feedback is the difference between practice and experience.)
3. **Watch video with the sound off, then on** — practice on recorded conversations (meetings, interviews, debates): first read the faces, then check against what was actually said and what actually happened. This is the closest practical approximation of the laboratory method available in normal life.
4. **FACS certification (optional, for serious use)** — 100+ hours of study plus the final test (Section 2.4). Worth it only if facial coding is part of your actual job (research, clinical, UX testing).
5. **Calibrate against the field gap** — periodically re-read Section 4.4. The skill you are building is *hypothesis generation about emotion*, not *truth detection*.

### 5.4 Resources

- **Books** — Paul Ekman, *Emotions Revealed* (2003): the accessible, non-technical statement of the science, the seven emotions, and the reading method; Ekman and Friesen, *Unmasking the Face* (1975): the classic field guide to the seven expressions with photo sets (still in print); Ekman, *Telling Lies* (4th ed., 2009): the deception book — including its careful caveats about what cues can and cannot do. *Verify: Emotions Revealed (Times Books, 2003) and Unmasking the Face (Prentice-Hall 1975) are standard, verified titles.*
- **Paul Ekman Group (PEG)** — paulekman.com: the FACS manual (2002 edition), METT, SETT, and online courses; the commercial home of the Ekman canon. *Verified as the official distributor; pricing/packaging current details not verified.*
- **David Matsumoto** — Humintell LLC (humintell.com): JACBART-based micro-expression tests, training, and cross-cultural research; Matsumoto is the leading independent researcher in this space. *Verified: Humintell is Matsumoto's company and sells micro-expression testing/training; treat specific offerings as current-approximate.*
- **Automated tools** — Affectiva, iMotions, Noldus FaceReader, OpenFace for AU detection (Section 2.5) — useful for *macro* expression research and product testing; not (yet) validated for live micro-expression reading.
- **Research literature** — Matsumoto and Hwang (2011) for training efficacy; Matsumoto et al. (2000) for JACBART; Frank et al. (2009) and the GAO reports for the skeptical/security side; Ekman and Friesen (1969, 1978) for the foundations.

---

## 6. Applications — Security, Law Enforcement, Clinical, Business, Technology

### 6.1 Security and law enforcement

- **Security screening** — the SPOT/BDA history (Section 4.3) is the definitive case study: behavior detection at airports was implemented at scale, never validated, and remains the cautionary tale for any "detect hostile intent from behavior" proposal. The evidence verdict is that **micro-expression-based screening does not work as a stand-alone security technology** — the base-rate problem alone is fatal.
- **Investigative interviewing** — the more defensible application: in a one-on-one interview (not mass screening), a trained interviewer can use facial signals — including micro and subtle expressions — as **leads for questioning**, the same way they use any inconsistency. This is consistent with modern interviewing frameworks such as the UK's **PEACE model** (Planning, Engage, Account, Closure, Evaluate — a non-accusatorial, information-gathering approach), which treats nonverbal cues as topics to explore rather than verdicts (see [body_language_lie_detection_guide.md](body_language_lie_detection_guide.md) for the PEACE discussion). The evidence supports "cues guide the questioning" — it does not support "cues prove the deception."
- **Courtroom and due diligence** — expert testimony on micro-expressions as deception evidence has been *excluded or heavily criticized* in multiple jurisdictions precisely because the science does not support individual-level truth judgments. The honest position for any professional setting: micro-expression observations are *interview tactics*, never *evidence*.

### 6.2 Clinical applications

- **Emotion-recognition deficits** — a substantial clinical literature documents impaired facial-emotion recognition in **autism spectrum disorder** (a core social-communication feature), **schizophrenia** (emotion perception deficits are among the best-replicated findings), and **mood disorders** (depression is associated with reduced recognition accuracy, particularly for positive emotions, and with a negativity bias). *Verify: these deficits are well-established in the clinical literature (e.g., Kohler et al. 2010 for schizophrenia meta-analyses; Harms, Martin and Wallace 2010 for autism; meta-analyses of depression and facial emotion recognition, e.g., Dalili et al. 2015). Micro-expression-specific deficit studies are thinner; the robust claims concern *facial emotion recognition generally*, of which micro-expression recognition is a subset.*
- **Therapy** — emotion-focused therapy and related approaches use moment-to-moment facial awareness ("the client's face just flashed anger") as clinical material; training tools (METT/SETT) have been used in social-skills and emotion-regulation interventions. The FACS system is also used in clinical research on pain expression and psychopathology.
- **For the manager** — the clinical relevance is indirect but real: it explains *why* some colleagues read faces easily and others genuinely cannot (neurotype, not hostility), which should temper both your reliance on your own face-reading and your assumptions about others'.

### 6.3 Business applications

- **Negotiations** — the flagship business use: watching the counterpart's face at hot spots (Section 1.4) — the moment a number is named, a term is rejected, a concession is made. The value is *directional*: a flash of fear at your BATNA statement says "that threat landed"; a flash of contempt at your offer says "that offer is an insult" — both are *hypotheses to probe*, not facts (Section 8; see also [vendor_management_guide.md](vendor_management_guide.md) for the negotiation context).
- **Interviews and hiring** — reading candidates for congruence between their words and their face ("you said you left on good terms — the flash of sadness/anger at the mention of your last manager suggests there's a story here worth one more question"). Used this way — as a question generator — it improves interviews; used as a pass/fail signal, it is pseudoscience with legal risk (hiring decisions must not rest on unvalidated nonverbal inferences).
- **Sales and customer work** — a customer's brief disgust flash at a price or a demo is the most actionable signal in selling: it arrives *before* the verbal objection, and the skilled response is to surface it ("you seem to have a concern about that number — what's your reaction?") rather than plow on.
- **Reading the room** — in meetings and stakeholder conversations, micro and subtle expressions are the early-warning system of the audience's true reaction before anyone speaks (see [communication_stakeholder_management_skills_guide.md](communication_stakeholder_management_skills_guide.md)); combined with the baseline and congruence work in [body_language_lie_detection_guide.md](body_language_lie_detection_guide.md), this is the everyday, low-stakes, high-value application — precisely because the stakes are low enough that a misread costs little and a probe question fixes it.

### 6.4 The honest business bottom line

Across all business applications the same pattern holds: **micro-expression reading is a conversational radar, not a decision engine.** It changes *what you ask*, not *what you conclude*. Every business application in this section works only when paired with (a) baseline calibration, (b) the hot-spot technique, (c) probe questions, and (d) explicit willingness to be wrong — the full method is in Section 7.

### 6.5 Technology — emotion AI, FER, and the EU AI Act

**Facial emotion recognition (FER)** — automated classification of emotion from faces (the technology layer under Affectiva, iMotions, etc., Section 2.5) — is a major branch of **affective computing** (computing that senses, models, and responds to emotion). The "emotion AI" market (FER plus voice-emotion, sentiment, and physiological sensing) has grown into a multi-billion-dollar industry serving advertising, automotive (driver monitoring), gaming, education, and HR-tech.

The regulatory environment has turned sharply against the most sensitive uses:

- **EU AI Act (Regulation (EU) 2024/1689)** — **Article 5(1)(f)** prohibits placing on the market or using AI systems that **infer emotions of natural persons in the workplace and in education institutions** (with narrow exceptions for medical and safety reasons). The prohibition has applied since **2 February 2025**. Outside workplace/education, emotion-recognition AI is treated as **high-risk** under Annex III (with transparency obligations), and the EU AI Act's provisions plus the GDPR create a high compliance bar for any emotion-inference product. *Verified: Article 5(1)(f) is the workplace/education emotion-recognition ban; the February 2025 applicability date is confirmed by multiple EU AI Act analyses. Verify note: the Act entered into force August 2024; Article 5 prohibitions applied from February 2, 2025 — "2024" in some summaries refers to the Act's adoption year, while the ban's *effect* date is 2025.*
- The scientific critique reinforces the regulatory one: FER's accuracy is **context-dependent and culturally variable** (the same face in different lighting, pose, or culture), the mapping from face to discrete emotion is exactly the contested universality assumption (Section 4.4), and there is no validated basis for using FER in hiring, insurance, or security decisions. For the AI/technical deep-dive, see the AI guides under [../technology/ai_llm/](../technology/ai_llm/).

**The technology bottom line:** automated facial coding is a mature measurement tool for *macro* expressions in controlled settings (research, UX, ad testing) and a growing but contested field for emotion *inference*; for *micro*-expression detection it remains experimental; and in the workplace it is now **prohibited in the EU**. The legal and scientific trajectories both point the same way: use the technology to *measure faces in research*, use *human judgment with probes* in conversation, and do not use either to *decide* about people.

---

## 7. Practical Guidance — How to Read Micro-Expressions

### 7.1 The method in five steps

The practical protocol, distilled from everything above. It is deliberately modest — it is a *conversation-improvement* method, not a *mind-reading* method:

1. **Watch the face — but selectively.** You cannot (and need not) monitor every muscle continuously. Fix your attention on the face during **hot spots** (Section 1.4): the moments when the stakes change — a number is named, a deadline mentioned, a rejection delivered, a question asked about money, a name raised. That is when leakage happens.
2. **Look for the mismatch — verbal vs facial.** The highest-value observation is *incongruence*: "he said *fine*, but his face flashed something else." You do not even need to name the emotion precisely — *"the face disagreed with the words"* is already a signal worth acting on. This is the congruence check, and it is the most robust practical skill in the entire field (see [body_language_lie_detection_guide.md](body_language_lie_detection_guide.md) for congruence across all channels — face, body, voice, words).
3. **Calibrate the baseline.** Before you read anyone, watch them *at rest* — their neutral face, their habitual smile, their normal speaking face. The baseline is your comparison point: a "smile" on a person who smiles all day means nothing; a flash of *anything* on a person whose face is normally still means something. Baselines are covered in detail in [body_language_lie_detection_guide.md](body_language_lie_detection_guide.md) — the same discipline applies to the face.
4. **Name the emotion tentatively, in AU language if you can.** When you catch a flash, force yourself through the checklist: brows (up? down? together?), eyes (wide? tight? averted?), mouth (corners up? down? tight? open?), symmetry (both sides or one?). Then map to the nearest prototype (Section 3.2). The discipline of *describing before labeling* is what separates a reader from a projector.
5. **Convert the observation into a question — always.** The micro-expression is a hypothesis, and the hypothesis is tested by asking, never by concluding. Section 8 shows the full sequence.

### 7.2 The "hot spots" checklist

Where to point your attention in a business conversation:

- **Money moments** — any price, budget, discount, or cost figure.
- **Commitment moments** — "can you deliver by X?", "will you sign?", "is this final?"
- **Relationship moments** — mentions of specific people (a manager, a rival vendor, a stakeholder) — the face often reacts to the *person* more than to the words.
- **Contradiction moments** — your own statement that contradicts theirs ("interesting — our records show the opposite").
- **Silence after a question** — the face during the pause before an answer is often more honest than the answer.
- **The transition into and out of their prepared statements** — the mask goes up and comes down; leakage lives at the edges (Section 3.4).

### 7.3 Red flags vs over-reading — the pitfalls

The failure modes, with their antidotes:

- **Single-cue over-reading.** One flash, one conclusion — the cardinal sin. *Antidote:* require *converging evidence* (the same signal twice, or the signal plus a verbal inconsistency) before you even form a hypothesis, and even then treat it as a hypothesis.
- **Confirmation bias.** You see what you expect: if you believe the vendor is hiding something, every face movement becomes "confirmation." *Antidote:* actively seek the alternative reading ("what would this flash look like if the person were merely tired/thinking hard/annoyed by the lighting?"). Ask the disconfirming question out loud.
- **Cultural context.** Display rules differ (Section 3.3): a smile, an averted gaze, or a suppressed reaction means different things in different cultures — and in *your own* culture, different subcultures and workplaces. What is "shifty" in one setting is "respectful" in another. *Antidote:* know the person's display-rule context before reading the face; when unsure, ask.
- **Baseline neglect.** Reading a naturally expressive person as "leaking" or a naturally still person as "hiding." *Antidote:* Step 3 — always.
- **The emotion-to-deception leap.** The most damaging error: "flash of fear ⇒ lying." Fear can mean fear of not being believed, fear of the consequences of *truth*, fear of you, fear of the room. *Antidote:* re-read Section 1.5 until it is reflex.
- **Over-confidence from training.** METT gains are real — on laboratory stimuli. The training itself can make you *more* confident without making you more accurate in the field (a well-documented effect in deception-detection research generally). *Antidote:* keep score. Write down your "reads" in meetings and check them against outcomes; let your own hit rate, not your confidence, tell you how good you are.
- **The "reading" that becomes a self-fulfilling prophecy.** Treating someone as deceptive changes how you question them, which changes their behavior, which "confirms" your read. *Antidote:* use probes as genuine information-seeking, not as traps.

### 7.4 Micro-expressions in the meeting — realistic expectations

What you can honestly expect from this skill in professional life:

- **Negotiations:** you will catch more *directional* signals — "that landed," "that insulted," "that surprised" — at the hot spots, and you will convert them into better questions and better timing. You will not "know what they're thinking," and anyone who tells you otherwise is selling something.
- **Interviews:** you will generate better follow-up questions from congruence mismatches, and you will avoid the hiring-pseudoscience trap by *never* letting a face movement decide a hire.
- **Feedback and difficult conversations** ([team_lead_methodologies_guide.md](team_lead_methodologies_guide.md)): the highest-leverage use — noticing in real time that your feedback landed as shame or anger rather than as information, so you can adjust the delivery *during* the conversation instead of after it.
- **Meetings:** you will read the room's *valence* (engaged vs checked out, aligned vs resistant) earlier and more accurately — because room-reading is mostly macro and subtle expressions, not micros, and macro/subtle reading *is* reliable enough for everyday use.

The honest limit in one line: **in a business meeting, the face is a good *thermometer* and a bad *X-ray*.** Use it to sense temperature changes; do not use it to inspect organs.

### 7.5 Ethics — privacy, consent, and the "not a truth machine" rule

The ethical frame for using this skill:

- **It is a truth-free technique.** Micro-expression reading detects *emotion*, not *truthfulness*, and it must be presented that way — to yourself, to your team, and to anyone you assess. Any use that implies "we can tell when you're lying" (in hiring, security, or disputes) is both scientifically unsupportable and ethically indefensible.
- **Consent and asymmetry.** You may be able to read more than the other person knows they are showing. In negotiation and interview contexts this is a legitimate skill (like any observation); the line is crossed when it becomes surveillance (recording and machine-analyzing faces without consent — which the EU AI Act now prohibits in the workplace, Section 6.5) or when it is used to *decide* about people rather than to *communicate* with them.
- **The reciprocity rule.** The skill is most defensible when it is used to *improve the conversation* (surface concerns, adjust delivery, ask better questions) rather than to *extract advantage*. The difference is visible to the other person, and it changes whether the skill reads as "great listener" or as "creepy."
- **The professional context.** In banking/enterprise settings, the safe rule is: micro-expression observations inform *questions*, never *decisions* — never a hiring decision, never a vendor score, never a compliance judgment, never a dispute outcome. Decisions get evidence; observations get questions.

### 7.6 Field drills — five exercises to build the habit

Skill without practice is trivia, so here are five low-cost drills that build the Section 7.1 method into habit — all of them safe, ethical, and usable in an ordinary working week:

1. **The baseline drill (one week).** Pick one colleague you see daily. Before any meeting with them, note their neutral face and their two or three habitual expressions (their "default smile," their "thinking" face, their "annoyed" face). By day five you will catch yourself automatically noticing *deviations* from their baseline rather than generic expressions — which is the entire point of calibration.
2. **The congruence drill (every meeting).** Pick one moment per meeting and check: *did the face agree with the words?* Not "what emotion was it" — just agreement/disagreement. Log it. After a month you will have a personal data set on how often faces and words diverge in your world — and you will have trained the single most robust observation skill there is.
3. **The hot-spot drill (negotiations and 1:1s).** Before an important conversation, write down the two or three hot spots you expect (the price, the deadline, the name). During the conversation, deliberately fix your attention on the counterpart's face *only at those moments* — and resist scanning at other times. This is attention management as much as observation.
4. **The video drill (weekly, 15 minutes).** Watch a recorded interview, debate, or press conference with the sound off first. Name the emotions you see per speaker per topic (tentatively, two options allowed: "fear or surprise"). Then replay with sound and check against what was actually said and what actually happened. This is the closest home approximation of laboratory training, and the replay is the feedback that makes practice deliberate.
5. **The probe drill (every hot spot).** When you observe a face–word mismatch, force yourself to convert it into a question *in the moment* — "what's your honest reaction to that number?" — and then notice what happens. The skill being trained is not reading; it is *using the read to improve the conversation*. The probe that surfaces the real objection (Section 8.4) is the payoff of the whole method.

One drill discipline: keep a private log (a notebook page or a file) of reads → probes → outcomes, and review it monthly. Your own hit rate is the only honest calibration instrument you will ever have (Section 4.5), and the log is also the antidote to the over-confidence that training itself can induce.

---

## 8. Worked Example — A Contempt Flash in a Vendor Negotiation

### 8.1 The scenario

You are in the final round of a vendor negotiation (the full negotiation context lives in [vendor_management_guide.md](vendor_management_guide.md)). You have just rejected the vendor's counter-offer and restated your number — a firm, below-their-ask figure. The vendor's sales director — let's call him R — pauses, then says, smoothly and warmly: **"Yes, the price is fine. We can make that work."** The words are agreeable; the tone is cooperative; the smile is in place.

### 8.2 The observation

In the fraction of a second *before* the smile, and again at the *edge* of it, you catch a flash: **the left corner of R's mouth tightened and lifted slightly — a unilateral lip-corner movement — then vanished** under the smile. Timing: well under 1/5 of a second — a genuine micro, not a held expression. *FACS reading:* a unilateral **AU12/AU14** — the dimpler/lip-corner pattern — the signature of **contempt** (Section 3.2), and notably *asymmetrical*, which is the contempt tell.

Context that makes the observation meaningful: this is a **hot spot** (your rejected counter-offer + his response), and the flash came at the mask boundary (Section 3.4) — the moment his managed smile was being assembled.

### 8.3 The analysis — and the discipline

What the flash means, honestly:

- **The emotion:** contempt (tentatively — one flash, one observer, imperfect lighting).
- **What it does NOT mean:** that R is lying, that the price is secretly unacceptable, that he will breach the contract, that he is a bad person. Contempt at that moment could mean: "this number is ridiculous and I'm conceding under protest," *or* "I despise this negotiating style," *or* "I'm contemptuous of my own procurement team for conceding," *or* "I just thought of a competitor I hold in contempt," *or* simply a habitual facial tic misread at speed.
- **The honest logic:** *the micro-expression suggests an emotion was present; the probe tests the hypothesis.* That is the entire method.

### 8.4 The follow-up — probing, not accusing

Do **not** say "you look contemptuous." Do not conclude "the price is secretly fine-for-them and I should push harder." Both are over-readings. Instead, convert the observation into a question that invites the emotion to surface or dissolve:

- *"You said 'fine' — but I get the sense the number isn't sitting well with you. What's your honest reaction to it?"* (names the mismatch, not the emotion — gives him room)
- *"If we land here, will your team support it internally, or will this create friction on your side?"* (tests the *reason* — the most common contempt trigger in a vendor deal is internal pressure, not your number)
- *"What would make this number easier for you to own?"* (turns the probe into a path forward)

The probe's job is not to "catch" him — it is to *test the hypothesis* and *get the real information*: whether the deal is actually acceptable, whether implementation will be undermined, whether the relationship needs repair. A contempt flash that is probed well often produces the actual objection — which is worth more than any facial read.

### 8.5 The caution — one cue, one hypothesis, no conclusion

The full discipline of this example, in four rules:

1. **One cue ⇒ one hypothesis, never a conclusion.** The flash generates a question. That is its ceiling.
2. **Require convergence before weight.** If the probe *confirms* the tension ("actually, the number is tough for us on the support costs"), *then* the contempt flash earns its place as one data point among several (their words, their other cues, the commercial context). If the probe dissolves it ("no, genuinely fine — I was thinking about the timeline"), the flash is archived as noise.
3. **Check your baseline.** Had R shown the same unilateral lip movement during pleasant small talk earlier? If yes, the "contempt" reading loses most of its value — it may be his habitual expression. (You would have known this only if you did Step 3 of Section 7.1.)
4. **Never let the read outrun the evidence in decisions.** The negotiation outcome — the price, the terms, the relationship — must rest on the commercial facts and the contract, not on a 150-millisecond face movement. The face improved your *questioning*; it did not change the *deal*.

---

## 9. Summary and the Honest Verdict

### 9.1 The honest answer

**Micro-expressions are real.** The phenomenon is well-documented (Ekman and Friesen 1969; the FACS system since 1978; the cross-cultural universality evidence from the Fore studies and Matsumoto's JACBART program). Brief, involuntary, full-intensity facial expressions exist; they leak concealed emotions; trained observers and trained software can detect them in controlled conditions; and untrained observers mostly miss them.

**Reading them is hard.** Untrained recognition of micro-duration expressions hovers near chance; METT-style training produces large laboratory gains, but the field evidence — for live, high-stakes, real-world reading — is thin, and the one large-scale real-world program built on the idea (TSA SPOT) was never validated and was abandoned in everything but name. The skill is real; the *deployment* record is a warning.

**The critical limit: emotion ≠ deception.** A micro-expression tells you an emotion was present. It does not tell you the truth, the lie, the reason, or the person's character. Every misapplication of this field — from airport screening to courtroom testimony to hiring tools — is a violation of this single sentence.

### 9.2 Best practice, in one line

> **Watch the face, listen to the words, check the congruence, ask the questions — and never conclude from one flash.**

That sentence is the whole guide in miniature: the face is a signal channel (watch it, especially at hot spots), the words are the primary data (listen to them), congruence is the highest-value observation (does the face agree with the words?), the probe question is the only legitimate output (test the hypothesis), and the "one flash, no conclusion" rule is the ethical and scientific floor.

### 9.3 Micro-expressions and the AI — the future

Automated facial coding (Affectiva, iMotions, Noldus, OpenFace) is mature for macro-expression measurement in research and product testing, and the "emotion AI" industry continues to grow — while the regulatory and scientific climate tightens against its most sensitive uses (the EU AI Act's workplace/education emotion-recognition ban since February 2025, and the ongoing scientific critique of face-to-emotion inference). The near-term honest forecast:

- **Measurement improves, inference does not.** AU detection will keep getting better (pose, lighting, occlusion robustness); the *meaning* of an AU sequence will remain context-dependent and contested.
- **Micro-expression automation stays experimental.** Detecting 100 ms flashes reliably from video is a hard computer-vision problem with slow progress and no validated field deployments.
- **The decision boundary stays human — and regulated.** In the EU, workplace emotion inference is prohibited; everywhere, the scientific basis for *deciding* from faces is absent. The AI future of this field is *assisted human conversation* — real-time visual cues feeding better questions — not *automated judgment*. For the technology deep-dive, see [../technology/ai_llm/](../technology/ai_llm/).

### 9.4 The final word

**Micro-expressions are a window, not a truth machine.** They open a small, real view onto what another person feels in a moment — a view worth having, worth training for, and worth using in every negotiation, interview, and difficult conversation you will ever run. But a window shows you a room's light, not its contents: it tells you an emotion was there, not why, not what it means, and not what to do. The skilled reader uses the window to look — and then asks. The unskilled reader uses it to conclude — and is wrong, predictably, with real costs. This guide's purpose is to make you the first kind: the reader who sees the flash, honors its limits, and turns it into the one question that matters.

### 9.5 FAQ — the five questions everyone asks

- **"Can micro-expressions tell me if someone is lying?"** No. They can tell you an emotion was present. Emotion ≠ deception (Section 1.5), and no study shows reliable lie detection from faces. The honest answer to "is he lying?" is: gather evidence, not expressions.
- **"Should I buy the micro-expression training course?"** Yes, if you want the skill as a *conversational* tool: METT/SETT training reliably improves emotion recognition of brief expressions (Section 4.1, 5.1), and the price is modest. No, if you expect it to make you a human lie detector or a negotiation super-weapon — it will not, and the course material will not tell you that; this guide just did.
- **"Why did the TSA program fail if the science is real?"** The science of *emotion leakage* is real; the application to *screening millions of people for rare hostile intent* is mathematically and practically hopeless — vanishing base rates produce overwhelming false positives (Section 4.3, 4.4). The failure was not "micro-expressions don't exist"; it was "micro-expressions cannot do what SPOT asked them to do."
- **"Is automated emotion recognition the future of this field?"** For *measurement* (research, UX, animation) yes — AU detection is mature (Section 2.5). For *inference about people* (hiring, security, workplace monitoring) the trajectory is the opposite: scientifically contested and now legally prohibited in EU workplaces (Section 6.5).
- **"What is the single most useful thing I can take from this guide?"** The probe: when the face and the words disagree, ask — "what's your honest reaction?" — and listen. One good question, asked at a hot spot, is worth more than a lifetime of watching for flashes.

---

## 10. Glossary

- **Micro-expression** — a very brief (≈1/25 to 1/5 second), involuntary, full-intensity facial expression that leaks a concealed emotion; the core phenomenon of this guide.
- **Macro-expression** — the normal, visible facial expression lasting roughly 1/2 to 4 seconds; the everyday managed or unmanaged face.
- **Subtle expression** — a low-intensity or partial facial expression (only part of the face, or faint), typically indicating a suppressed or nascent emotion; distinct from a micro (which is full-intensity but brief).
- **Leakage** — the involuntary transmission of concealed information through nonverbal channels (face, body, voice); the micro-expression is the facial form (Ekman and Friesen, 1969).
- **Hot spot** — a moment in a conversation when emotional stakes spike (money, commitments, people, contradictions), where leakage is most likely to occur.
- **FACS (Facial Action Coding System)** — the standard system for describing facial movement in terms of Action Units; published by Ekman and Friesen in 1978, revised 2002.
- **Action Unit (AU)** — a FACS-coded individual facial muscle movement (e.g., AU4 brow lowerer, AU6 cheek raiser, AU12 lip corner puller); 44 AUs in the original system, 44–46 in the 2002 revision.
- **Duchenne smile** — the genuine enjoyment smile: AU6 (cheek raise / crow's feet) + AU12 (lip corners up); the eye muscle component that most people cannot contract voluntarily.
- **Display rules** — culturally learned norms governing when, to whom, and how intensely emotions may be expressed (suppression, masking, intensification, qualification).
- **Masking** — replacing the true expression with a different, socially acceptable one (e.g., a smile over sadness); micro-expressions often leak at the edges of a mask.
- **Suppression (neutralization)** — hiding an emotion behind a neutral face; the display-rule strategy that produces subtle expressions.
- **Seven universal emotions** — anger, fear, sadness, disgust, surprise, contempt, happiness; the Ekman set with the strongest cross-cultural recognition evidence (six original + contempt, added in the 1990s).
- **Anger** — the emotion of the narrowed, hard face: brows down (AU4), eyes wide with tightened lids (AU5+AU7), lips tight (AU23/24).
- **Fear** — the emotion of the opened, pulled-back face: brows raised and drawn together (AU1+2+4), eyes very wide (AU5), lips stretched (AU20).
- **Sadness** — the emotion of the drooping face: inner brows up (AU1), brows down (AU4), lip corners down (AU15), chin raised (AU17).
- **Disgust** — the emotion of the turning-away face: nose wrinkled (AU9), upper lip raised (AU10).
- **Surprise** — the emotion of the opening face: brows raised (AU1+2), eyes wide (AU5), jaw dropped (AU26/27); the shortest-lived and easiest-to-fake macro emotion.
- **Happiness** — the emotion of the Duchenne smile: lip corners up (AU12) plus cheek raise and crow's feet (AU6); the genuine-smile signature.
- **Contempt** — the emotion expressed by the unilateral lip-corner movement (AU12/AU14); the only universal emotion whose expression is typically asymmetrical; the "sneer."
- **METT (Micro Expression Training Tool)** — Ekman's self-instructional training program with pre/post tests; the most-studied micro-expression training instrument.
- **SETT (Subtle Expression Training Tool)** — the companion tool training recognition of low-intensity/partial (subtle) expressions.
- **Paul Ekman** — the psychologist who, with Wallace Friesen, discovered/described micro-expressions (1969), built FACS (1978), and led the cross-cultural universality research; author of *Emotions Revealed* and *Telling Lies*.
- **Emotions Revealed** (2003) — Ekman's accessible book on the seven emotions and reading faces; the standard popular starting point.
- **David Matsumoto** — psychologist who extended the cross-cultural universality research and built JACBART, the standard instrument for measuring micro-expression recognition; also a leading critic of over-claimed applications.
- **Universality** — the claim, supported by Ekman's and Matsumoto's cross-cultural studies, that the basic emotional expressions are recognized across cultures (including the Fore people of Papua New Guinea).
- **Fore people** — the Papua New Guinea culture studied by Ekman in the 1960s–70s; their correct matching of Western-expression photos is the classic universality evidence.
- **JACBART** — Japanese and Caucasian Brief Affect Recognition Test (Matsumoto et al., 2000): brief-presentation emotion recognition test used to measure micro-expression reading.
- **TSA SPOT** — Screening of Passengers by Observation Techniques: the US airport behavior-detection program (2007–, renamed BDA in 2016) whose effectiveness the GAO repeatedly found unvalidated; the cautionary tale of the field.
- **GAO** — US Government Accountability Office; its 2010 and 2013 reports (GAO-10-763, GAO-14-159) found SPOT's effectiveness not demonstrated.
- **Emotion AI / affective computing** — technology that senses, models, and responds to human emotion; facial emotion recognition (FER) is its facial branch.
- **FER** — Facial Emotion Recognition: automated classification of emotion from facial images/video.
- **Affectiva / iMotions / Noldus (FaceReader) / OpenFace** — leading automated facial-coding platforms (commercial and open-source) detecting AUs and emotions from video.
- **EU AI Act** — Regulation (EU) 2024/1689; its Article 5(1)(f) prohibits AI that infers emotions in the workplace and in education (in force since February 2025), with emotion recognition elsewhere high-risk.
- **Recognition** — correctly identifying which emotion an expression shows (the measured skill in micro-expression research).
- **Baseline** — a person's habitual, at-rest facial/behavioral pattern, against which deviations are judged.
- **Congruence** — consistency between channels (words, face, body, voice); mismatches are the highest-value observations.
- **Emotion vs deception** — the critical distinction of this guide: a micro-expression evidences an emotion, never truthfulness; "emotion ≠ deception."

---

## 11. References and Further Reading

**Primary sources (verified):**

- Ekman, P., & Friesen, W. V. (1969). *Nonverbal leakage and clues to deception.* Psychiatry, 32(1), 88–106. doi:10.1080/00332747.1969.11023575.
- Ekman, P., Sorenson, E. R., & Friesen, W. V. (1969). *Pan-cultural elements in facial displays of emotion.* Science, 164(3875), 86–88.
- Ekman, P., & Friesen, W. V. (1971). *Constants across cultures in the face and emotion.* Journal of Personality and Social Psychology, 17(2), 124–129.
- Ekman, P., & Friesen, W. V. (1978). *Facial Action Coding System: A technique for the measurement of facial movement.* Consulting Psychologists Press.
- Ekman, P., Friesen, W. V., & Hager, J. C. (2002). *Facial Action Coding System (2nd ed.).* Research Nexus / Paul Ekman Group.
- Matsumoto, D., LeRoux, J., Wilson-Cohn, C., et al. (2000). *A new test to measure emotion recognition ability: Matsumoto and Ekman's Japanese and Caucasian Brief Affect Recognition Test (JACBART).* Journal of Nonverbal Behavior, 24(3), 179–209.
- Matsumoto, D., & Hwang, H. S. (2011). *Evidence for training the ability to read microexpressions of emotion.* Motivation and Emotion, 35(2), 181–191.
- Frank, M. G., Maccario, C. J., & Govindaraju, V. (2009). *Behavior and security.* In P. Zelkowitz (Ed.), Protecting Airline Passengers in the Age of Terrorism. ABC-CLIO.
- GAO (2010). *Aviation Security: Efforts to Validate TSA's Passenger Screening Behavior Detection Program.* GAO-10-763.
- GAO (2013). *Aviation Security: TSA Should Limit Future Funding for Behavior Detection Activities.* GAO-14-159.
- European Union (2024). *Regulation (EU) 2024/1689 (AI Act), Article 5(1)(f) — prohibited emotion-inference systems in the workplace and education.*

**Popular / practitioner (verified titles):**

- Ekman, P. (2003). *Emotions Revealed: Recognizing Faces and Feelings to Improve Communication and Emotional Life.* Times Books.
- Ekman, P., & Friesen, W. V. (1975). *Unmasking the Face: A Guide to Recognizing Emotions from Facial Clues.* Prentice-Hall.
- Ekman, P. (2009). *Telling Lies: Clues to Deceit in the Marketplace, Politics, and Marriage* (4th ed.). W. W. Norton.
- Ekman, P. (1980). *The Face of Man: Expressions of Universal Emotions in a New Guinea Village.* Garland STPM Press. (Documentation of the Fore fieldwork.)
- Paul Ekman Group — paulekman.com: FACS manual (2002), METT, SETT, online training.
- Humintell (David Matsumoto) — humintell.com: JACBART-based micro-expression testing and training.

**Skeptical / critical (for balance):**

- Weinberger, S. (2010). *Airport security: Intent to deceive?* Nature, 465, 412–415. (Behavior detection's scientific problems.)
- Barrett, L. F. (2017). *How Emotions Are Made: The Secret Life of the Brain.* Houghton Mifflin Harcourt. (The constructed-emotion critique of the basic-emotions framework.)
- ACLU reporting and litigation records on the TSA SPOT/BDA program (profiling complaints and effectiveness challenges).

**Companion guides in this series (see also):** [body_language_lie_detection_guide.md](body_language_lie_detection_guide.md) (baselines, cue clusters, PEACE model, NLP eye patterns — the full body-language frame of which this guide is the facial deep-dive) · [communication_stakeholder_management_skills_guide.md](communication_stakeholder_management_skills_guide.md) (conversation, stakeholder and EI skills) · [team_lead_methodologies_guide.md](team_lead_methodologies_guide.md) (feedback and difficult conversations) · [vendor_management_guide.md](vendor_management_guide.md) (negotiation and due diligence) · [../technology/ai_llm/](../technology/ai_llm/) (AI and automated facial-recognition technology).

**A note on verification.** Every factual claim in this guide was checked against at least one independent source at the time of writing. Items marked *verified* in the text (the Ekman & Friesen 1969 citation, FACS dates and AU numbers, the seven-emotion list, the METT/SETT tools, the GAO SPOT reports, the EU AI Act Article 5(1)(f) prohibition and its February 2025 applicability, JACBART, the Duchenne AU6+AU12 signature, the 100+ hour FACS certification benchmark) are corroborated by the primary or official sources listed above. Items marked with an explicit *verify note* or described as approximate (exact recognition-accuracy percentages, training-gain magnitudes, micro frequency in natural conversation, current BDA operational status, current training prices) are flagged because published figures vary across studies or change over time; treat them as ballparks, not constants. Where the literature is contested (the basic-emotions framework itself, field transfer of training, automated FER reliability), both sides are cited so the reader can weigh them.

**Suggested citation.** Liu, J. S. (2026). *Micro-Expressions: A Comprehensive Guide.* Management & Leadership Series, research repository (Cymbal Bank, Singapore).

---

*Guide complete. The honest takeaway, once more: micro-expressions are a window, not a truth machine — watch the face, listen to the words, check the congruence, ask the questions, and never conclude from one flash.*
