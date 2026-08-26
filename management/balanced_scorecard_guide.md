# The Balanced Scorecard: The Strategy on One Page — A Comprehensive Guide

*A deep dive into the Balanced Scorecard (BSC) — the performance-measurement framework that turned strategy into a dashboard: the Kaplan–Norton 1992 origin, the four perspectives, the strategy maps, the measures (lagging vs leading), the cascading, the implementation, the banking applications, the criticisms (Jensen, Norreklit), the ESG-era evolution, and a full worked example on a Cymbal Bank.*

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Series:** Management / Leadership (management/) — this is the **dedicated deep-dive** on the Balanced Scorecard; the study-guide genre follows [../technology/grokking_system_design_companion_guide.md](../technology/grokking_system_design_companion_guide.md)
> **Verification discipline:** every factual claim below is either (a) **verified** against a primary or authoritative source (HBR, HBS Press, the academic journals, the Balanced Scorecard Institute) in this edition, (b) **cross-referenced** to a sibling guide in this repo that already verified it, or (c) explicitly **flagged** as unverified/standard attribution. Nothing is fabricated.

---

## How This Guide Fits the Repo

The BSC appears throughout this repository as an anchor framework; this guide is the deep-dive that ties those appearances together and goes beyond them:

| Sibling guide | Where the BSC appears | What this guide adds |
|---|---|---|
| [mba_body_of_knowledge_guide.md](mba_body_of_knowledge_guide.md) | §3.6 — **verifies the origin**: Kaplan & Norton, "The Balanced Scorecard—Measures That Drive Performance," *HBR* 70(1), Jan–Feb 1992, pp. 71–79 | The full framework: perspectives, maps, measures, criticism, banking worked example |
| [it_strategy_guide.md](it_strategy_guide.md) | §8 — the BSC as the **IT-value framework** (IT scorecard: financial/customer/process/learning) + a Cymbal Bank worked example (§8.2–8.3, "Step 7 — Value") | Cross-referenced heavily in §3 (maps), §4 (leading/lagging), §7 (banking) and §10 (the worked example) |
| [strategic_management_guide.md](strategic_management_guide.md) | §4.1 — BSC as the strategy-implementation framework + a regional-bank scorecard (§4.1.1) | The deep treatment: why the four perspectives, how strategy maps and cascading actually work, what the critics say |
| [business_case_development_guide.md](business_case_development_guide.md) | The KPI / benefit-realisation discipline (ex-ante case → ex-post tracking) | §4 (measures) and §6 (implementation) close the loop from *justification* to *ongoing measurement* |
| [organizational_behavior_guide.md](organizational_behavior_guide.md) | The alignment/culture angle: measurement systems shape behaviour | §2 (perspectives), §6 (implementation: the people side), §8 (gaming critique) |
| [management_consulting_skills_guide.md](management_consulting_skills_guide.md) · [mckinsey_approach_guide.md](mckinsey_approach_guide.md) | The consulting framing: hypotheses, structure, board-readable deliverables | §6 (implementation process) and §10 (the worked example as a consulting-style deliverable) |
| [the_managers_path_research.md](the_managers_path_research.md) · [grow_team_guide.md](grow_team_guide.md) | People measurement: engagement, capability, development | §2 (learning & growth) and §5 (personal scorecards) |
| [../banking/capital_markets_architecture_guide.md](../banking/capital_markets_architecture_guide.md) · [../banking/credit_agricole_software_systems_guide.md](../banking/credit_agricole_software_systems_guide.md) · [../banking/hsbc_software_systems_guide.md](../banking/hsbc_software_systems_guide.md) | The banking context: business lines, systems landscape, the CIB operating model | §7 (banking scorecards) and §10 (the Cymbal Bank worked example) |
| [../technology/grokking_system_design_companion_guide.md](../technology/grokking_system_design_companion_guide.md) | The study-guide structure pattern | The overall architecture of this guide |

**How to read it:** §1–§4 are the theory (what the BSC is, its four lenses, its maps, its measures). §5–§6 are the machinery (cascading and implementation). §7–§9 are context (banking practice, the critics, the evolution). §10 is a full worked example in the familiar Cymbal Bank context. §11 is the one-page summary — the BSC's own promise: *the strategy on one page*. The glossary and verification notes close the file.

---

## Table of Contents

1. [The BSC Overview](#1-the-bsc-overview)
2. [The Four Perspectives](#2-the-four-perspectives)
3. [The Strategy Maps](#3-the-strategy-maps)
4. [The Measures](#4-the-measures)
5. [The Cascading](#5-the-cascading)
6. [The Implementation](#6-the-implementation)
7. [The Banking Scorecards](#7-the-banking-scorecards)
8. [The Criticisms](#8-the-criticisms)
9. [The Evolution](#9-the-evolution)
10. [The Worked Example: A Cymbal Bank Scorecard](#10-the-worked-example-a-cymbal-bank-scorecard)
11. [The Summary: The Strategy on One Page](#11-the-summary-the-strategy-on-one-page)
12. [The Glossary](#12-the-glossary)
13. [Verification Notes and Sources](#13-verification-notes-and-sources)

---

## How to Study This Guide

- **New to the BSC:** read §1 → §2 → §3 → §4 in order (the idea, the lenses, the map, the measures), then jump straight to §10 (the worked example) and come back for machinery and critique.
- **Preparing for a strategy/performance discussion at work:** §2, §5 and §10 are the practical core; §8 gives you the questions to ask when someone presents a scorecard.
- **Building an IT or unit scorecard:** start from [it_strategy_guide.md](it_strategy_guide.md) §8 (the IT-value frame) and use this guide's §3.5 (build a map first) and §4.4 (measure hygiene) as the method.
- **Revision:** the bold claims in each section are the examinable core; the tables are the one-page summaries of each section; §11 compresses the whole guide; §13 tells you what is verified versus flagged.
- **Reading time:** ~45 minutes cover-to-cover; §10 + §11 alone in ~10 minutes.

---

## 1. The BSC Overview

### 1.1 The Kaplan–Norton 1992 Origin (verified — deepened, not re-derived)

The origin is **already verified in this repo** — [mba_body_of_knowledge_guide.md](mba_body_of_knowledge_guide.md) §3.6 (its frameworks chapter, marked *verified*): Robert S. Kaplan (then Arthur Lowes Dickinson Professor of Accounting at Harvard Business School) and David P. Norton (president of Nolan, Norton & Co., the Massachusetts IT consulting firm he cofounded) published **"The Balanced Scorecard—Measures That Drive Performance" in *Harvard Business Review* 70, no. 1, January–February 1992, pp. 71–79**. This edition re-confirmed the citation directly against HBR's own archive page ("From the Magazine (January–February 1992)", reprint 92105) and the full reprint text — so the origin is **doubly verified** and the space here is spent *deepening* it, per the series convention.

What the 1992 article actually argued (from the primary text):

- **"What you measure is what you get."** The opening line is not a slogan; it is the mechanism. A measurement system strongly affects the behaviour of managers and employees, so choosing measures *is* choosing behaviour.
- **The problem it attacked:** traditional financial accounting measures (ROI, earnings-per-share) "worked well for the industrial era" but give **misleading signals for continuous improvement and innovation** — the activities the information-age competitive environment demands.
- **The non-negotiable complement:** "The balanced scorecard includes financial measures that tell the results of actions already taken. And it complements the financial measures with operational measures on customer satisfaction, internal processes, and the organization's innovation and improvement activities—operational measures that are the **drivers of future financial performance**." The BSC was never "drop the financials" — it was "stop flying on one instrument."
- **The cockpit metaphor:** managing a complex organization is like flying an airplane: "Reliance on one instrument can be fatal." The scorecard is the dials and indicators in the cockpit.
- **The provenance:** the scorecard emerged from "a year-long research project with **12 companies** at the leading edge of performance measurement" — a 1990 multi-company study run with the Nolan Norton Institute (its 1991 report preceded the article; cross-ref [it_strategy_guide.md](it_strategy_guide.md) §8.2, which verifies the same provenance).
- **The early-technology lineage (flagged):** per the Wikipedia treatment of the BSC's history, a "corporate scorecard" design at Analog Devices by Art Schneiderman (1987) and the French engineers' *tableau de bord* ("dashboard") tradition were antecedents; Schneiderman participated in the 1990 study. This is secondary-source attribution — flagged, not re-verified against primary documents this edition.

The origin story therefore has three beats this guide will develop across the next sections:

1. **1992 — a measurement system** (four perspectives; this guide §1–§4);
2. **1996 — a strategic management system** ("Using the Balanced Scorecard as a Strategic Management System," *HBR*, first published January–February 1996; four management processes; §5–§6);
3. **2000 — strategy maps** ("Having Trouble with Your Strategy? Then Map It," *HBR*, September–October 2000; §3), then the *Strategy-Focused Organization* (2000), *Alignment* (2006) and *The Execution Premium* (2008) book-era extensions (§9).

### 1.2 What the BSC Is — and Is Not

| The BSC is… | The BSC is not… |
|---|---|
| A **performance-measurement system**: a balanced set of financial and non-financial measures against targets | A replacement for financial reporting — it is financial measures' *complement* (verified, 1996 article: "The scorecard wasn't a replacement for financial measures; it was their complement") |
| A **strategic management system**: the 1996 article's core claim — it links long-term strategy with short-term actions | A budgeting tool — though business planning is one of its four management processes |
| A **communication device**: it puts "the strategy in a bottle" (1996 article, the oil-company executive's wish) | A once-a-year reporting exercise — it is designed for periodic strategic review (feedback and learning) |
| A **guard against suboptimization**: "the scorecard lets managers see whether improvement in one area may have been achieved at the expense of another" (1992) | A measure-proliferation exercise — "The balanced scorecard forces managers to focus on the handful of measures that are most critical" (1992; the "kill another tree program" warning) |
| A **top-down reflection of mission and strategy** (1993 article's first characteristic) | A bottom-up collection of whatever metrics are easy to compute |

The 1993 follow-up, "Putting the Balanced Scorecard to Work" (*HBR*, September–October 1993), sharpened the definition with four distinctive characteristics (verified via the 1996 reprint's Further Reading annotations): it is (1) a **top-down reflection** of the company's mission and strategy; (2) **forward-looking**; (3) an **integration of external and internal measures**; and (4) a **focusing device** for motivating and implementing breakthrough performance.

### 1.3 The Overview Table

| Aspect | Description |
|---|---|
| **Name** | Balanced Scorecard (BSC) — "balanced" because it holds financial and non-financial, external and internal, lagging and leading measures in one frame |
| **Creators** | Robert S. Kaplan (HBS accounting professor) and David P. Norton (Nolan, Norton & Co. / later Palladium) |
| **Origin** | *HBR* 70(1), January–February 1992, pp. 71–79 — "The Balanced Scorecard—Measures That Drive Performance" (verified, doubly: sibling §3.6 + HBR archive this edition) |
| **Provenance** | 1990–91 multi-company research project with 12 companies at the leading edge of performance measurement (Nolan Norton Institute; cross-ref [it_strategy_guide.md](it_strategy_guide.md) §8.2) |
| **Core move** | One financial measure set is insufficient; view performance through four perspectives: financial, customer, internal process, learning & growth (verified, 1992 text) |
| **Core promise** | "What you measure is what you get" — the measure set *drives* behaviour, so design it to drive strategy (verified, 1992 text) |
| **Core metaphor** | The airplane cockpit: many instruments, none sufficient alone; "reliance on one instrument can be fatal" (verified, 1992 text) |
| **Unit of analysis** | The strategy — the scorecard is strategy translated into objectives, measures, targets and initiatives |
| **Canonical texts** | 1992 HBR (origin) → 1993 HBR (four characteristics) → 1996 HBR (strategic management system) + 1996 book → 2000 HBR (strategy maps) + 2000 book → 2004 *Strategy Maps* → 2006 *Alignment* → 2008 *The Execution Premium* (each verified/flagged in §9) |
| **Companion discipline** | Strategy maps (the "what" of strategy), cascading (the "who"), benefit realisation ([business_case_development_guide.md](business_case_development_guide.md)), OKRs (the lighter-weight cousin — cross-ref [strategic_management_guide.md](strategic_management_guide.md) §4.2) |
| **Signature output** | The scorecard report itself — one page of objectives, measures, targets and initiatives across four perspectives: *the strategy on one page* |

### 1.4 The Canon in One Paragraph

The BSC canon is unusually tight — two authors, one idea, five HBR articles and four books. **1992**: measures that drive performance (the four perspectives). **1993**: putting it to work (top-down, forward-looking, integrated, focusing). **1996**: using it as a strategic management system (translating the vision, communicating and linking, business planning, feedback and learning). **2000**: strategy maps — "Having Trouble with Your Strategy? Then Map It" — and *The Strategy-Focused Organization* (the five principles). **2004**: *Strategy Maps: Converting Intangible Assets into Tangible Outcomes* (the full map methodology; standard attribution — flagged). **2006**: *Alignment: Using the Balanced Scorecard to Create Corporate Synergies* (verified via the 2007 reprint's author bio). **2008**: *The Execution Premium* (verified via the Wikipedia treatment of the BSC's history). Every section below anchors to one of these.

### 1.5 Why the BSC Won — and What It Competes With

The BSC is one of the most successful management ideas of the last half-century, and the tool surveys are the evidence: it became a staple of the Bain Management Tools repertoire in the 1990s–2000s (historical adoption commonly cited at roughly half of surveyed firms — **flagged**: Bain figures were not retrievable this edition, as the search backend was intermittently down) and remains a standard strategy-implementation instrument (the 2GC 2020 survey reported via Wikipedia: **88% of respondents use the BSC for strategy-implementation management and 63% for operational management** — flagged, reported via Wikipedia). Its competition in the goal-setting space, and how the repo's sibling guide positions them:

| Framework | Origin | Structure | BSC relationship |
|---|---|---|---|
| **BSC** | Kaplan & Norton, HBR 1992 (verified) | Four perspectives, strategy map, measures/targets/initiatives, cascade | This guide |
| **OKRs** | Intel / Andy Grove; popularised by Google (Doerr) — standard attribution | Objectives (qualitative) + 2–4 Key Results (quantitative) per level | Lighter-weight cousin: less causal machinery, faster cycle; cross-ref [strategic_management_guide.md](strategic_management_guide.md) §4.2 |
| **Hoshin Kanri** | Japanese policy deployment (Toyota et al.) | True North, breakthrough objectives, catchball, X-matrix, PDCA | Same alignment ambition with a negotiation-based cascade; cross-ref [strategic_management_guide.md](strategic_management_guide.md) §4.3 |
| **MBO** | Drucker, 1954 (standard attribution) | Top-down objectives + appraisal | The pre-BSC ancestor; the BSC fixed its two flaws: no non-financial balance, no causal logic |

The BSC's durable differentiators are exactly what the competitors lack: the **four-perspective balance**, the **cause-and-effect map** (the testable theory of the business), and the **cascade to personal scorecards**. That combination is why it outlived MBO and absorbed the OKR generation's energy without being displaced by it.

---

## 2. The Four Perspectives

### 2.1 Verification

The four perspectives are **verified at the primary source** (the 1992 article's own exhibit, "The Balanced Scorecard Links Performance Measures", reproduced in the reprint this edition consulted). The scorecard "provides answers to four basic questions":

1. **How do customers see us?** — the *customer* perspective
2. **What must we excel at?** — the *internal* perspective (internal business processes)
3. **Can we continue to improve and create value?** — the *innovation and learning* perspective
4. **How do we look to shareholders?** — the *financial* perspective

The 1996 article confirmed the settled names: the scorecard "supplemented traditional financial measures with criteria that measured performance from three additional perspectives—those of **customers, internal business processes, and learning and growth**" (verified, 1996 reprint text). The sibling guides carry the same verified list: [mba_body_of_knowledge_guide.md](mba_body_of_knowledge_guide.md) §3.6 and [it_strategy_guide.md](it_strategy_guide.md) §8.2–8.3 (financial, customer, internal process, learning & growth), and the Balanced Scorecard Institute's current exposition (consulted this edition) keeps the same four with the fourth renamed "Organizational Capacity" in its *Nine Steps to Success* methodology — a deliberate practitioner evolution, flagged as such (see §9).

### 2.2 The Financial Perspective — "How do we look to shareholders?"

The scorecard's top layer and ultimate destination. In the 1992 scheme the financial perspective keeps the shareholder view honest: for a for-profit enterprise the strategy must *end* in financial performance. The 1992 article's answer to the question "How do we look to shareholders?" is deliberately generic because the measures depend on the strategy — the canonical examples are ROI/ROE, earnings, revenue growth, cash flow, cost-to-income. Kaplan and Norton's own later work (and the BSI's current guidance, consulted this edition) puts the financial perspective **at the top of the strategy map** for commercial organisations, because for them it is the end of the value chain — a placement that *reverses* for public-sector and non-profit scorecards (see §2.7).

### 2.3 The Customer Perspective — "How do customers see us?"

The 1992 article is explicit that customer concerns "tend to fall into four categories: **time, quality, performance and service, and cost**." The customer perspective forces the company to translate a mission like "be number one in delivering value to customers" into *specific* measures of what the company delivers: lead time (time to meet the customer's needs), quality (defect level as *perceived and measured by the customer*), performance-and-service, and price. The 1992 case material (ECI, the semiconductor company) shows the discipline: ECI discovered each major customer defined "reliable, responsive supply" differently, built a database of customer-defined factors, and redefined "on time" to match customer expectations — external data, gathered from outside the firm.

### 2.4 The Internal Process Perspective — "What must we excel at?"

Customer measures "must be translated into measures of what the company must do internally to meet its customers' expectations" (verified, 1992 text): "excellent customer performance derives from processes, decisions, and actions occurring throughout an organization." The internal-perspective measures are the *machinery* — cycle time, quality, throughput, straight-through processing, operational efficiency, risk outcomes. Note the direction of logic: internal excellence is defined *by* the customer value proposition, not by internal convenience — a point §3's strategy maps make visually.

### 2.5 The Learning and Growth Perspective — "Can we continue to improve and create value?"

The innovation-and-learning perspective (1992 name; "learning and growth" settled by 1996) covers "the organization's innovation and improvement activities" — the capacity that makes future financial performance possible. This is the perspective that most directly captures the **intangible assets** the information age runs on: employee skills, information systems, climate/culture, and the ability to launch new products and improve processes. In the causal chain it is the *root*: it drives internal processes, which drive customer outcomes, which drive financial results (cross-ref [it_strategy_guide.md](it_strategy_guide.md) §8.3's learning-and-growth row: modern skills, attrition, capability maturity, architecture obsolescence — the "future capability stock").

### 2.6 The Perspectives Table

| Perspective | The question it answers (1992, verbatim) | Example measures (1992 canon + banking/IT practice) |
|---|---|---|
| **Financial** | "How do we look to shareholders?" | ROI, ROE, revenue growth, cash flow, cost-to-income ratio, EVA, earnings growth |
| **Customer** | "How do customers see us?" | Market share, customer retention/acquisition, satisfaction (CSAT/NPS), on-time delivery, customer-defined quality, time-to-respond |
| **Internal process** | "What must we excel at?" | Cycle time, quality rate/defect levels, first-pass yield, operational efficiency, STP rate, NPL ratio (banking), release frequency (IT) |
| **Learning & growth** | "Can we continue to improve and create value?" | Employee satisfaction/engagement, training hours, % revenue from new products, innovation rate, skills coverage, attrition |

(Cross-ref [strategic_management_guide.md](strategic_management_guide.md) §4.1, which carries the same four-perspective table with banking-flavoured example metrics.)

### 2.7 The Balance, and the Fifth-Perspective Question (flagged where applicable)

- **"Balanced" is a design principle, not arithmetic.** The balance is between *financial and non-financial*, *external (shareholders, customers) and internal (processes, capabilities)*, *lagging outcomes and leading drivers* (§4). There is no requirement to weight the perspectives equally.
- **The public-sector inversion (verified, BSI this edition):** mission-driven organisations put **customer/stakeholder on top** and financial *stewardship* beneath it — "financial stewardship is not the end of the value chain for that type of organization; stakeholder satisfaction is." Perspective names may change (Stewardship, Organizational Capacity, People/Knowledge/Technology) while the underlying focus stays.
- **The fifth-perspective debate (flagged):** the criticism literature (see §8.4) argues the four perspectives omit wider stakeholders — suppliers, regulators, society, the environment. The sustainability/ESG era responds by *adding* perspectives or extending the map (see §9.3). The 1992–2008 canon itself never added a fifth perspective; later work (e.g. the 2019 "Intelligent Design of Inclusive Growth Strategies" article) folds societal outcomes into the framework instead.

### 2.8 The Four Questions as a Management Discipline (verified, 1992 company examples)

The 1992 article's company cases show the questions operating as a *discipline*, not a taxonomy:

- **Customer measures come from customers, not from internal convenience.** A computer manufacturer that wanted to be the competitive leader in customer satisfaction measured *competitive rankings* obtained from an outside firm talking directly to customers, plus the percentage of revenue from third-party relationships (partnerships). A producer of very expensive medical equipment whose customers demanded high reliability built two customer-based metrics: **equipment up-time percentage** and **mean-time response to a service call**. A semiconductor company asked each major customer to rank it against comparable suppliers on quality, delivery time, and price performance — and when it discovered it ranked in the middle, managers made improvements that moved it to the top (all verified, 1992 text).
- **The questions are asked at every level.** "How do customers see us?" is as meaningful for a branch, a desk, or an IT service as for the whole firm — which is why the four questions survive the cascade (§5) unchanged in spirit.
- **The questions force the trade-offs into the open.** "What must we excel at?" cannot be answered without deciding what the firm will *not* excel at — the same focusing discipline as the strategy map's value-proposition choice (§3.3).

The practical takeaway for a Solution Architect or a line manager: before adding any metric to any dashboard, ask which of the four questions it answers. If it answers none, it is noise — the 1992 article's "kill another tree program" warning in one sentence.

---

## 3. The Strategy Maps

### 3.1 Verification

The strategy map's canonical statement is **Kaplan & Norton, "Having Trouble with Your Strategy? Then Map It," *Harvard Business Review*, September–October 2000** — verified this edition against HBR's own archive (the article page confirms authors, title, and "From the Magazine (September–October 2000)"; the reprint's opening uses the campaign-map metaphor: a general cannot communicate a strategy without a map of the terrain). The companion book, *The Strategy-Focused Organization* (2000), carried the "Strategic Linkage Model" / strategy-map idea into the book canon (verified via the Wikipedia treatment of the BSC's history, which dates the concept to the 2000 book and notes the earlier Scandinavian Olve et al. 1999 parallel — flagged). The 2004 book *Strategy Maps: Converting Intangible Assets into Tangible Outcomes* (HBS Press) is the standard attribution for the full methodology (flagged — not re-verified against the book itself this edition). The Balanced Scorecard Institute's current definition (consulted this edition) is consistent: "A strategy map is a simple graphic that shows a logical, cause-and-effect connection between strategic objectives (shown as ovals on the map)."

### 3.2 The Cause-and-Effect Logic (verified)

The map's spine is **cause-and-effect across the four perspectives, bottom-up**:

**Learning & Growth → Internal Process → Customer → Financial**

The 1992 article already gestured at this ("By combining the financial, customer, internal process and innovation, and organizational learning perspectives, the balanced scorecard helps managers understand, at least implicitly, many interrelationships"); the 1996 article made the causal claim explicit and operational — "because of the cause-and-effect linkages inherent in the scorecard framework, changes in one component of the system reinforced earlier changes made elsewhere" — and the 2000 article turned the implicit chain into a drawn artifact. The chain's meaning: investments in people, systems and climate (L&G) enable better processes (internal), which deliver the value proposition (customer), which produces financial results. The map is therefore the answer to the 1996-era question: *"How do L&G investments drive financial results?"* (cross-ref [strategic_management_guide.md](strategic_management_guide.md) §4.1's closing note on strategy maps).

### 3.3 Anatomy of a Map (verified, BSI + 2000 article + practice)

- **Ovals and arrows:** strategic objectives are drawn as ovals, grouped by perspective layer; **arrows** show the cause-and-effect relationships, read bottom-up.
- **Scale:** a typical map holds **12–18 strategic objectives** across the four layers (BSI, consulted this edition) — the same "handful of measures" discipline as the scorecard itself.
- **The value proposition sits at the customer layer.** The map forces an explicit choice of how the firm wins: cost leadership, product leadership, or customer intimacy (the classic three value propositions in the 2004 book's methodology; standard attribution — flagged).
- **Financial on top (for-profit):** "most for-profit companies put the financial perspective on top because their end goal is to make more money" (BSI, verified this edition); public sector swaps the top two layers (§2.7).
- **Themes:** real maps are often organised into strategic themes (e.g. "operational excellence", "growth", "digital transformation") that each trace their own causal thread from L&G up to financial.

### 3.4 The Map Table

| Layer (bottom → top) | Typical strategic objectives | The causal link it contributes |
|---|---|---|
| **Learning & Growth** (the root) | Build digital skills; improve leadership bench; modernise platforms; foster innovation culture | Capability is the *cause*: you cannot run better processes without better people, systems and climate |
| **Internal Process** | Accelerate loan decisions; raise STP rate; strengthen risk controls; shorten time-to-market | Process excellence *produces* the customer value proposition |
| **Customer** | Improve NPS; grow SME wallet share; improve digital adoption; on-time delivery | Customer outcomes *convert* process excellence into revenue |
| **Financial** (the outcome) | ROE > target; cost-to-income < target; revenue growth; capital efficiency | Financial results *validate* the whole chain — the strategy's end state |

The 1992 four questions reappear as the layers' *why*: L&G answers "can we continue to improve and create value?", internal answers "what must we excel at?", customer answers "how do customers see us?", financial answers "how do we look to shareholders?" — the map is the four questions wired together with arrows.

### 3.5 Building a Map — the Practical Sequence (verified, BSI + 1996 practice)

1. **Agree the destination:** mission, vision, and the handful of strategic themes (the 1996 "translating the vision" process — §6.1).
2. **Choose the value proposition** at the customer layer (cost / product leadership / customer intimacy).
3. **Draft objectives per layer** (12–18 total), phrased as continuous-improvement verbs ("improve…", "accelerate…", "reduce…").
4. **Draw the arrows** between objectives, bottom-up, asking at every link: "does this genuinely drive that?" — the test that the Norreklit critique (§8.2) says most organisations answer too glibly.
5. **Pressure-test with cross-functional teams** — BSI recommends cross-functional development "to expand the set of viewpoints informing strategy formulation and encourage buy-in."
6. **Derive the scorecard from the map:** every objective on the map gets a measure, a target, and (usually) an initiative — the map is the "what", the scorecard is the "how measured".

### 3.6 The Five Principles of the Strategy-Focused Organization (2000 book — verified name, via Wikipedia)

The 2000 book *The Strategy-Focused Organization* (verified via the Wikipedia treatment of the BSC's history) distilled the strategy-map era into **five principles** — the standard list, reproduced here with the repo's lens:

1. **Translate the strategy to operational terms** — the strategy map (§3).
2. **Align the organization to the strategy** — enterprise scorecard and cascading (§5).
3. **Make strategy everyone's everyday job** — communication, personal scorecards, incentives (1996 process 2).
4. **Make strategy a continual process** — feedback and learning, the review cadence (1996 process 4).
5. **Mobilize change through executive leadership** — the 1992 finding that senior managers must be in the design.

### 3.7 A Mini Strategy Map, Drawn in Text (Meridian Bank, §10)

```
Financial        [RoTE > 12%]   <-  [Cost-to-income < 55%]  <-  [NBI growth +6%]
                     ^                     ^                       ^
Customer      [Client NPS +15]  <-  [Wallet share +3pts]    <-  [Digital adoption 70%]
                     ^                     ^                       ^
Internal       [Loan decision <48h] <- [STP 85%]        <-  [Early-warning 100%]
                     ^                     ^                       ^
L&G        [Digital certification 60%] <- [Platform modernisation] <- [HiPo bench 90%]
```

Read bottom-up: certification, platforms and bench produce the process outcomes (decision time, STP, risk coverage); those produce the customer outcomes (NPS, wallet share, digital adoption); those produce the financial outcomes. Every arrow is a hypothesis the quarterly review tests (§6.1, process 4) — and every arrow is exactly what Norreklit (2000) demands be questioned (§8.2).

---

## 4. The Measures

### 4.1 Verification: Lagging vs Leading

The lagging/leading distinction is **verified at the primary source**. The 1992 article: "The balanced scorecard includes financial measures that tell the **results of actions already taken** [lagging outcomes]. And it complements the financial measures with operational measures … that are the **drivers of future financial performance** [leading drivers]." The 1996 reprint's Editor's Note says the same thing in one line: "These nonfinancial metrics are so valuable mainly because they **predict future financial performance** rather than simply report what's already happened." The sibling repo treatment is consistent: [it_strategy_guide.md](it_strategy_guide.md) §8.5 ("Mix leading and lagging. Lagging indicators … confirm the past; leading indicators … predict the future. A scorecard of only lagging indicators is a rear-view mirror") — cross-ref.

- **Lagging (outcome) measures** report results after the fact: profit, revenue, NPS, NPL ratio. They are the *destination* — but by the time a lagging measure moves, the causes are history.
- **Leading (driver) measures** predict future outcomes: training hours, skills coverage, release frequency, early-warning triggers, first-call resolution. They are the *steering wheel* — but a leading measure with no proven link to an outcome is just activity.

The 1996 article's cause-and-effect discussion adds the essential warning: the *driver* measures only matter if the causal link to outcomes holds, and validating that link "can take months or years" — meanwhile "managers' assessment of strategic impact may have to rest on subjective and qualitative judgments." That humility is exactly what the Norreklit critique (§8.2) pushes further.

### 4.2 Outcome Measures vs Performance Drivers (verified, 1996)

The 1996 article's vocabulary distinguishes **outcome measures** (e.g. profitability, market share, customer satisfaction) from the **performance drivers** of those outcomes (e.g. the retraining, information systems, and product introductions that produce them). A scorecard needs both: outcomes alone are a scoreboard with no playbook; drivers alone are activity with no score. The 1996 exhibit "How One Company Linked Measures from the Four Perspectives" shows the full chain — the scorecard as a *theory of the business* written in measures.

### 4.3 The Measures Table

| Type | Role in the scorecard | Examples (general / banking / IT) |
|---|---|---|
| **Lagging (outcome)** | Confirms the past; the scoreboard; answers "did we get there?" | ROI, ROE, revenue growth, NPS, retention, market share · bank: cost-to-income, NPL ratio, RoTE · IT: availability, cost per transaction |
| **Leading (driver)** | Predicts the future; the steering wheel; answers "will we get there?" | Training hours, skills coverage, innovation rate, % revenue from new products · bank: loan-decision cycle time, STP rate, early-warning coverage · IT: release frequency, architecture obsolescence index |
| **Financial** | The ultimate outcomes for a for-profit (1992: "How do we look to shareholders?") | Profit, cash flow, ROE, EVA, cost-to-income |
| **Non-financial** | The drivers and early signals the financials cannot see (customer, process, capability) | CSAT, quality, cycle time, engagement, % digitised |

### 4.4 Measure Hygiene (verified 1992 + BSI + practice)

- **Few, critical, strategic.** "Companies rarely suffer from having too few measures. More commonly, they keep adding measures … The balanced scorecard forces managers to focus on the handful of measures that are most critical" (1992). The BSI's current guidance adds the KPI test: objective, time-bound, focused on results not activities, widely understood.
- **Every measure must trace to a strategy-map objective.** If a metric cannot be traced to an objective on the map, it does not belong (cross-ref [it_strategy_guide.md](it_strategy_guide.md) §8.5's rule: "if a metric cannot be traced, drop it").
- **Targets and initiatives complete the line.** A measure without a target is decoration; a target without an initiative is a wish (this is the objective–measure–target–initiative row structure used throughout §10).
- **Guard against suboptimization.** "Even the best objective can be achieved badly" (1992): time-to-market can improve by managing introductions *or* by releasing only incrementally different products; output can rise on easy low-margin mix. The scorecard's four-perspective simultaneity is the guard.
- **The benefit-realisation loop (cross-ref).** Business-case targets (ex-ante, [business_case_development_guide.md](business_case_development_guide.md)) must meet post-implementation review and ongoing benefit tracking — the ex-post half of the same discipline ([it_strategy_guide.md](it_strategy_guide.md) §8.5).

### 4.5 Target-Setting and Stretch (verified, 1996 + practice)

- **Targets are the ambition layer.** A measure without a target is a thermometer without a patient: it reports without committing. The 1996 business-planning process uses the scorecard's "ambitious goals" as "the basis for allocating resources and setting priorities" — targets *drive* the budget, not the other way round.
- **Stretch targets need drivers underneath.** The 1996 logic: agree the outcome measures, then "identify the most influential 'drivers' of the desired outcomes and set milestones for gauging the progress they make with these drivers." A stretch NPS target without named drivers (RM training, portal fixes, onboarding time) is a wish.
- **Calibration is an annual event, not a one-off.** The 1996 feedback-and-learning process makes target recalibration part of strategic review: disconfirming evidence may mean adjusting "the quantitative relationship among the strategic measures" — i.e. the target itself, or the causal link (§8.2's critique made operational).
- **Benchmarks beat guesses.** The 1992 semiconductor case shows the technique: external customer rankings gave the company a calibrated target ("rank in the middle" → "move to the top") rather than an invented number.

---


## 5. The Cascading

### 5.1 Verification

Cascading is **verified at the 1996 primary source and the BSI methodology**:

- The 1996 article's second management process, **"communicating and linking"**, is cascading in its original form: "When a scorecard is disseminated up and down the organizational chart, strategy becomes a tool available to everyone. As the high-level scorecard **cascades down** to individual business units, overarching strategic objectives and measures are translated into objectives and measures appropriate to each particular group. Tying these targets to individual performance and compensation systems yields **'personal scorecards'**" (verified, 1996 reprint text).
- The 1996 article also shows the *upward* direction: "The balanced scorecard … should also be communicated **upward** in the organization—to corporate headquarters and to the corporate board of directors," so business units can "quantify and communicate their long-term strategies to senior executives."
- The Balanced Scorecard Institute formalises the tiers (consulted this edition): "**Cascading a balanced scorecard means to translate the corporate-wide scorecard (Tier 1) down to first business units, support units or departments (Tier 2) and then teams or individuals (Tier 3).** The end result should be focus across all levels of the organization that is consistent."
- The book-length treatment is Kaplan & Norton, *Alignment: Using the Balanced Scorecard to Create Corporate Synergies* (HBS Publishing, **2006** — verified via the 2007 reprint's author bio), which extends alignment from the unit level to the *enterprise* level: an enterprise scorecard and strategy map that articulate the "enterprise value proposition" — how the whole creates value beyond the sum of units (verified via the HBS product page for *Alignment*, consulted this edition).

### 5.2 How Cascading Works

- **Tier 1 — enterprise/corporate:** the firm-wide strategy map and scorecard. Everything below must trace to it.
- **Tier 2 — business units, support units, departments:** each unit derives its own scorecard from the enterprise objectives that *its* work serves (e.g. a Markets unit, a Trade Finance unit, an IT department — see [it_strategy_guide.md](it_strategy_guide.md) §8 for the IT-unit variant). Objectives translate, they do not copy: the corporate "improve cost-to-income" becomes the IT unit's "reduce unit cost per transaction" and the branch network's "reduce branch cost per customer".
- **Tier 3 — teams and individuals:** team scorecards and "personal scorecards" tied to performance and compensation (1996). The individual sees *their* productivity's contribution to the strategy — the mechanism behind the 1996 claim that "individual employees understand how their own productivity supports the overall strategy."
- **The alignment test:** the cascaded scorecards must be *consistent* in direction (BSI: "focus across all levels … that is consistent") while remaining *relevant* at each level — a Tier-3 scorecard that copies Tier-1 measures wholesale is a poster, not a cascade.
- **Bottom-up input is part of the design:** the 1996 article describes a company that deliberately involved three layers of management — the senior group set financial and customer objectives, then "mobilized the talent and information in the next two levels" to formulate the internal-process and learning-and-growth objectives that would drive them. Cascading is negotiation, not dictation (the same spirit as Hoshin Kanri's catchball — cross-ref [strategic_management_guide.md](strategic_management_guide.md) §4.3).

### 5.3 The Cascading Table

| Level (tier) | Scorecard | Example objectives (banking flavour) | Alignment question answered |
|---|---|---|---|
| **Tier 1 — Enterprise / corporate** | Corporate strategy map + scorecard (12–18 objectives) | ROE > 12%; cost-to-income < 55%; #1 NPS in SME segment; NPL < 2.5% | "What is the firm's strategy, in measures?" |
| **Tier 2 — Business unit / support unit** | Unit scorecard derived from Tier 1 (translate, don't copy) | Markets: revenue per desk, VaR limit adherence; Trade Finance: STP rate, onboarding time; IT: unit cost per transaction, availability; Branch network: cost per customer, digital adoption | "Which enterprise objectives does *this unit* serve, and how is that measured?" |
| **Tier 3 — Team / individual** | Team scorecards + personal scorecards (tied to performance and compensation, 1996) | RM: portfolio NPS, cross-sell per client; Developer: release frequency, change-failure rate; Ops team: STP rate, error rate | "What does *my* work contribute to the strategy?" |

### 5.4 Cascading Pitfalls

- **The poster cascade:** Tier-2/3 scorecards that restate Tier-1 measures without translation — alignment theatre (cross-ref [it_strategy_guide.md](it_strategy_guide.md) §8.4's "reporting exercise nobody reads" warning).
- **The attribution trap:** a unit cannot own an outcome it does not control. Cascading must allocate *drivers* to units and keep *outcomes* at the level where accountability is real ("Attribution is a governance act, not a measurement act" — [it_strategy_guide.md](it_strategy_guide.md) §8.4).
- **Local optimisation:** a unit hitting its own targets at the expense of another unit's (the 1992 suboptimization warning, now between units). The enterprise map exists precisely to expose this.
- **Incentive distortion:** tying pay to a few measures invites gaming — the Jensen critique (§8.3) bites hardest exactly here.

### 5.5 A Worked Tier-2 Cascade (Meridian Bank, §10 — translation, not copying)

The enterprise objectives from §10.2, translated for three units:

| Enterprise objective (Tier 1) | Global Markets (Tier 2) | Trade Finance (Tier 2) | IT / Operations (Tier 2) |
|---|---|---|---|
| RoTE > 12% | Desk-level RoTE; VaR limit adherence; funding cost per desk | NBI per trade; cost per transaction | Unit cost per transaction; run-the-bank vs change-the-bank split (§8.4 in [it_strategy_guide.md](it_strategy_guide.md)) |
| Client NPS +15 | e-trading adoption; execution quality (slippage) | Onboarding time; exception rate | System availability; MTTR; SLA attainment |
| Loan decision < 48h | — | Decision time for trade facilities | Workflow engine latency; credit-scoring model uptime |
| STP 85% | Algo STP rate | STP rate (documentary + open account) | Integration latency (T+0 data); change-failure rate |
| Digital certification 60% | Quant/algo skills coverage | Trade-platform skills | Cloud/AI capability maturity; architecture obsolescence index |

Every Tier-2 row answers: *which enterprise objective do I serve, and how is *my* contribution measured?* — the alignment question from §5.3.

---

## 6. The Implementation

### 6.1 Verification: The Four Management Processes (1996)

Implementation in the Kaplan–Norton canon is the **1996 article's four management processes**, verified this edition against the reprint text (R0707M):

1. **Translating the vision.** "By relying on measurement, the scorecard forces managers to come to agreement on the metrics they will use to operationalize their lofty visions." The Metro Bank case is the proof: 25 senior executives all "agreed on the words of the strategy" — "to provide superior service to targeted customers" — yet "each one had a different definition of superior service and a different image of the targeted customers"; building the scorecard forced consensus (verified, 1996 text).
2. **Communicating and linking.** The cascade (§5) plus education: brochures, town meetings, bulletin boards, groupware — and *linking rewards to performance measures*.
3. **Business planning.** Integrating strategic planning with budgeting — otherwise, in the 1996 quote, long-term planning is where "the rubber meets the sky." The scorecard's ambitious targets become the basis for resource allocation and initiative prioritisation.
4. **Feedback and learning (strategic learning).** Periodic reviews evaluate not just "did we meet budget?" but *"is the strategy working, and if not, why?"* — using the scorecard's causal model to test the theory of the business; disconfirming evidence triggers double-loop learning (the 1996 article's closing argument: strategic learning "is what distinguishes the balanced scorecard").

The 1996 article also verifies the *pace*: National Insurance built its system "step-by-step over **30 months**, with each step representing an incremental improvement," reconsidering each process two or three times before the system stabilised. Implementation is a campaign, not a project.

### 6.2 The Implementation Table

| Step | What happens (verified source) | Typical output | Repo cross-ref |
|---|---|---|---|
| **1. Translate the vision** | Executives convert mission statements into agreed objectives and measures (1996; the engineering-construction CEO's "what am I supposed to do?" phone call is the canonical failure mode) | Agreed strategy statement; draft strategy map (§3) | [strategic_management_guide.md](strategic_management_guide.md) §2.3 (strategy implementation) |
| **2. Communicate and link** | Cascade Tier 1 → Tier 2 → Tier 3; educate; tie rewards (1996; BSI tiers) | Unit/team/personal scorecards (§5) | [organizational_behavior_guide.md](organizational_behavior_guide.md) (alignment/culture); [grow_team_guide.md](grow_team_guide.md) (people measures) |
| **3. Business plan** | Budgets, capital allocation and initiatives aligned to scorecard targets (1996) | Integrated budget + initiative portfolio | [business_case_development_guide.md](business_case_development_guide.md) (ex-ante case ↔ ex-post tracking) |
| **4. Feedback and learn** | Periodic strategic review; test the causal model; double-loop learning (1996) | Revised strategy or recalibrated measures | [mckinsey_approach_guide.md](mckinsey_approach_guide.md) (hypothesis-driven review) |
| **5. Sustain (the 30-month lesson)** | Iterate the four processes until the system stabilises (1996, National Insurance) | An embedded management system, not a report | [management_consulting_skills_guide.md](management_consulting_skills_guide.md) (change discipline) |

### 6.3 Success Factors and Failure Modes (verified + flagged)

- **Senior-manager involvement is non-negotiable (verified, 1992 text):** "the project participants found that they could not implement the balanced scorecard without the involvement of the senior managers who have the most complete picture of the company's vision and priorities. This was revealing because most existing performance measurement systems have been designed and overseen by financial experts."
- **Design by the users, not by consultants (flagged, via the Wikipedia treatment of the BSC's history):** Arthur Schneiderman's 1999 article "Why Balanced Scorecards Fail" (Journal of Strategic Performance Measurement, January) argued many early failures came from scorecards "designed remotely by consultants" — managers who were not involved did not trust the design and did not engage. Secondary-source attribution; the 1999 article itself was not consulted this edition.
- **The metric-tracing rule (cross-ref):** every measure traces to a strategy-map objective; untraceable measures get dropped ([it_strategy_guide.md](it_strategy_guide.md) §8.5).
- **Evidence of effectiveness (verified via the Wikipedia treatment):** Ittner, Larcker & Randall (2003), "Performance implications of strategic performance measurement in financial services firms," *Accounting, Organizations and Society* 28(7): 715–741, is the canonical empirical study demonstrating a link between BSC-style measurement use and better decision making/performance in financial-services firms — the closest the literature gets to a bank-specific validation.

### 6.4 The Implementation Checklist (the 1996 processes made operational)

- [ ] **Vision translated:** the executive team has converted the vision into agreed objectives and measures — not just words (the Metro Bank test, §6.1).
- [ ] **Map drawn and owned:** 12–18 objectives, four layers, arrows agreed by a cross-functional team (§3.5).
- [ ] **Scorecard derived:** every map objective has a measure, a target, and a named initiative; every measure traces to a map objective (§4.4).
- [ ] **Cascade designed:** Tier-2 units have translated scorecards; Tier-3 personal scorecards exist and are linked to performance and compensation (§5).
- [ ] **Planning integrated:** budgets and capital allocation follow scorecard targets — no "rubber meets the sky" separation (§6.1, process 3).
- [ ] **Review cadence set:** periodic (monthly/quarterly) sessions that test the causal model, not just the budget — strategic learning (§6.1, process 4).
- [ ] **Leadership locked in:** senior managers with "the most complete picture of the company's vision" are in the design and the review (§6.3, 1992 finding).
- [ ] **Patience budgeted:** expect two or three iterations of the four processes before the system stabilises — the National Insurance 30-month lesson (§6.1).

---

## 7. The Banking Scorecards

### 7.1 Verification

Banking is not an afterthought in the BSC canon — the **1996 article's running example is a bank**: **Metro Bank** ("not its real name"), "the result of a merger of two competitors," whose strategy was "to provide superior service to targeted customers" (verified, 1996 reprint text — the full Metro Bank narrative is in §2.1/§6.1). The 1996 article's "How One Company Built a Strategic Management System" exhibit tracked National Insurance (an insurer), and the causal-chain exhibit is drawn from the same banking/insurance material. The empirical anchor is Ittner, Larcker & Randall 2003 (financial services firms, §6.3). The repo's own banking treatment mirrors this: [strategic_management_guide.md](strategic_management_guide.md) §4.1.1 carries a regional-bank scorecard ("Cymbal Bank–style") and [it_strategy_guide.md](it_strategy_guide.md) §8.2–8.3 builds the IT scorecard for a bank — cross-ref both heavily.

### 7.2 Why the BSC Fits Banking

- **Intangibles dominate.** A bank's value comes from client relationships, risk skill, data, and employee expertise — exactly the assets the learning-and-growth and internal-process perspectives exist to measure. The 1996 framing ("ability to exploit intangible assets has become far more decisive") was written for the information age generally; banks are its purest case.
- **Regulation makes outcomes multi-dimensional.** Capital, liquidity, conduct and NPL outcomes cannot be read off a single profit number; the BSC's four-perspective frame accommodates risk and compliance as internal-process outcomes (cross-ref [../banking/risk_management_models_guide.md](../banking/risk_management_models_guide.md) lightly).
- **The branch network is a natural cascade.** Tier 2/3 cascading maps cleanly onto business lines → regions → branches → relationship managers.
- **Long causal chains.** Training a trader or digitising onboarding pays off years later — precisely the lagging/leading problem the BSC was built for.

### 7.3 The Banking Table

| Perspective | Banking objective (from the sibling regional-bank example and the canon) | Banking measures | Cross-ref |
|---|---|---|---|
| **Financial** | Improve profitability; grow net banking income; optimise cost base | ROE / RoTE, revenue growth, cost-to-income ratio, NPL ratio | [strategic_management_guide.md](strategic_management_guide.md) §4.1.1; [../banking/universal_banking_model_guide.md](../banking/universal_banking_model_guide.md) |
| **Customer** | Increase loyalty; grow the SME segment; improve digital experience | NPS, retention, market share, digital adoption, cross-sell per client | [../banking/capital_markets_architecture_guide.md](../banking/capital_markets_architecture_guide.md) (client coverage models) |
| **Internal process** | Accelerate loan approvals; strengthen risk management; improve operational efficiency | Loan decision time, STP rate, NPL ratio, early-warning coverage, KYC/onboarding time | [it_strategy_guide.md](it_strategy_guide.md) §8.3 (process row); [../banking/credit_agricole_software_systems_guide.md](../banking/credit_agricole_software_systems_guide.md) |
| **Learning & growth** | Build digital capabilities; enhance the leadership bench; foster innovation culture | % staff digitally certified, succession coverage, training hours, idea rate, attrition | [the_managers_path_research.md](the_managers_path_research.md); [grow_team_guide.md](grow_team_guide.md) |

### 7.4 The Branch-Level Scorecard (verified pattern, banking practice)

A branch is a Tier-2/3 unit: its scorecard *translates* enterprise objectives. Financial: branch revenue, cost per customer. Customer: local NPS, retention, digital adoption in the branch's catchment. Internal: teller/queue time, STP rate, error rate, compliance findings. Learning & growth: staff certification, product knowledge. The discipline is the same as §5.3 — the branch must be able to point at every one of its measures and name the enterprise objective it serves.

### 7.5 The Metro Bank Story, Retold (verified, 1996 article)

Metro Bank is the BSC canon's own banking case — "not its real name," the result of a merger of two competitors, whose senior executive group believed it had agreed the strategy: "to provide superior service to targeted customers." Research had revealed **five basic market segments** among existing and potential customers, each with different needs. The moment of truth came while formulating the customer-perspective measures: although the **25 senior executives agreed on the words** of the strategy, "each one had a different definition of superior service and a different image of the targeted customers." The exercise of developing operational measures forced the 25 to clarify the meaning of the strategy statement: they ultimately agreed to stimulate revenue growth through new products and services, settled on the **three most desirable customer segments**, and built scorecard measures for the specific products, services and relationships each segment required. The scorecard then "highlighted gaps in employees' skills and in information systems that the bank would have to close" to deliver the chosen value propositions — the learning-and-growth layer of the map, derived from the customer layer.

The banking lessons in one paragraph: (1) strategy agreement on *words* is not agreement on *meaning* — measures force meaning; (2) segmentation is a prerequisite for the customer perspective, and the customer perspective is what forces segmentation; (3) the capability gap (skills + systems) is discovered by working top-down from customer commitments, never by guessing bottom-up; (4) a merged bank needs the scorecard precisely because two cultures measure two different things.

---

## 8. The Criticisms

### 8.1 Verification: The Two Named Critics

Both canonical critiques are **verified this edition**:

- **Hanne Norreklit (2000):** "The balance on the balanced scorecard – a critical analysis of some of its assumptions," *Management Accounting Research* **11(1): 65–88**, doi:10.1006/mare.1999.0121 — verified via the Wikipedia treatment's reference list this edition. This is the **causality critique**.
- **Michael C. Jensen (2001):** "Value maximisation, stakeholder theory, and the corporate objective function," *European Financial Management* **7(3): 297–318**, doi:10.1111/1468-036x.00158 (a companion version appeared in the *Journal of Applied Corporate Finance* the same year) — verified via the Wikipedia treatment's reference list this edition. This is the **gaming / no-overall-score critique**. (Note: the series brief calls the title "Value Maximization and the Corporate Objective Function"; the verified titles are the two above.)

The Wikipedia treatment (consulted this edition) groups BSC criticism into **three areas**: (1) **lack of rigour** — no formal validation and, notoriously, no citations in the early articles (an absence Norreklit flagged); (2) **lack of an overall score** — the BSC is a list of metrics with no unified score or clear recommendation (Jensen 2001); (3) **failure to reflect all stakeholder needs** — a financial-stakeholder bias carried from the original US-commercial orientation.

### 8.2 Norreklit's Causality Critique (2000)

Norreklit's core attack is on the **cause-and-effect assumptions** that §3 built on:

- **Correlation is not causation.** The maps assert that L&G → internal → customer → financial, but the framework offers no evidence the links hold; the 1996 article itself concedes validating causal links "can take months or years."
- **Time-lag problems.** The perspectives' effects do not line up in the neat bottom-up order: learning today may show up in financials on a very different timescale (or not at all), and feedback loops run in both directions.
- **Balance is asserted, not argued.** Norreklit questions whether the four perspectives are actually "balanced" — the model's own logic privileges the financial perspective, which sits atop the chain as the end state.
- **Lack of scholarly scaffolding.** The 1992 and 1993 articles cite no prior work — a legitimacy problem for an academic-origin framework (per the Wikipedia treatment, "an absence noted, for example, by Norreklit").
- **The practical sting:** if the causal model is unvalidated, the scorecard's strategic-learning machinery (§6.1, process 4) is testing an unproven theory — managers may "learn" the wrong lessons.

### 8.3 Jensen's Gaming Critique (2001)

Jensen's critique comes from his value-maximization camp and has two prongs, both verified at the level of the cited paper's argument as summarised in the literature:

- **No overall score → no objective function.** "The balanced scorecard does not provide an overall score or a unified view of performance with clear recommendations: it is simply a list of metrics that managers have to interpret before deciding upon appropriate interventions" (Wikipedia's summary of the Jensen 2001 criticism). For Jensen, a management system that cannot score the whole cannot be held accountable for the whole.
- **Gaming.** Where multiple metrics compete for attention and rewards, managers optimise the measured game: they hit the visible targets (or manage the measures themselves) rather than create value — "what you measure is what you get" turned perverse. Jensen's broader argument is that stakeholder-ish, multi-objective scorecards invite exactly this: no single objective (shareholder value maximization) to arbitrate trade-offs, so effort flows to whatever is measured and rewarded. (The specific "gaming" language is part of Jensen's wider critique of scorecard-based incentive systems in that paper and its JACF companion; the *verified* core, per the citation summary, is the no-overall-score argument. Flagged for precision.)

### 8.4 The Other Criticisms

- **Stakeholder omission (verified, Wikipedia treatment):** the four perspectives privilege shareholders and customers; suppliers, regulators, employees-as-stakeholders, and society are under-represented — a charge the public-sector scorecard, the 3rd-generation BSC, the Public Sector Scorecard and UN Results-Based Management explicitly try to answer (verified names, via Wikipedia).
- **Non-profit mismatch (verified, Wikipedia treatment):** "the balanced scorecard does not address important aspects of nonprofit strategy such as social dimensions, human resource elements, political issues and the distinctive nature of competition and collaboration in nonprofit settings."
- **Rigor and validity (verified, Wikipedia treatment):** no formal validation of the four-perspective choice (Flamholtz); measurement-design flaws (Malina, Lingle & Schiemann); the "why scorecards fail" literature (Schneiderman 1999).
- **Effectiveness evidence is thin (verified, Wikipedia treatment):** few reliable assessments; the notable exception is Ittner, Larcker & Randall 2003 in financial services (§6.3).

### 8.5 The Criticisms Table

| Critique | Source (verified) | The argument | The counter (from the canon and practice) |
|---|---|---|---|
| **Causality is unproven** | Norreklit 2000, *Management Accounting Research* 11(1): 65–88 | Correlation ≠ causation; time lags break the neat L&G→financial chain; balance is asserted, not argued | The 1996 article already concedes the lag and treats the causal model as a *hypothesis* to be tested in strategic review; the map's value is making the theory explicit and falsifiable |
| **No overall score / gaming** | Jensen 2001, *European Financial Management* 7(3): 297–318 | A metric list with no single objective invites gaming and evades accountability | Kaplan–Norton never claimed an overall score; the BSC is a *management* instrument, not an objective function — and the gaming risk is managed by balanced, traceable, few measures (§4.4) |
| **Lack of rigour / no validation** | Norreklit 2000; Flamholtz 2003; Schneiderman 1999 (all via Wikipedia treatment) | No citations in early articles; no formal validation of the four perspectives | Later-generation designs (2nd/3rd generation, Public Sector Scorecard, Nine Steps) add structure; the empirical literature (Ittner et al. 2003) shows positive performance association in financial services |
| **Stakeholder bias** | Wikipedia treatment (Andersen & Lawrie 2002; Kong 2010) | Four perspectives privilege financial stakeholders; public/non-profit needs under-served | The BSI's mission-driven scorecard inverts the map and renames perspectives (§2.7); ESG-era extensions add societal outcomes (§9.3) |
| **Implementation failures** | Schneiderman 1999; Malina & Selto 2001 (via Wikipedia treatment) | Consultant-designed, trust-less scorecards fail; users don't engage | The 1992 article itself insists on senior-manager involvement; §6.3's design-by-users rule |

### 8.6 The Defenders Respond

The canon's own answers to the critics are not afterthoughts — they are built into the 1996 article:

- **To Norreklit (causality):** the 1996 article *concedes the epistemological point in advance* — validating causal links "can take months or years," and interim assessment "may have to rest on subjective and qualitative judgments." The scorecard's answer is not to claim proven causation but to make the theory **explicit and falsifiable**: "getting managers to think systematically about the assumptions underlying their strategy is an improvement over the current practice of making decisions based on short-term operational results." A strategy map you can argue with beats an implicit strategy you cannot.
- **To Jensen (no overall score):** Kaplan–Norton never promised an overall score; they promised a *management instrument* — "the scorecard isn't a replacement for financial measures; it is their complement." The overall score, in their frame, is the financial perspective's lagging outcome; the non-financial perspectives exist to *explain and drive* it. The gaming risk is real (their own "what you measure is what you get" concedes the mechanism) and is answered by design: few measures, traceable to the map, balanced across perspectives (§4.4).
- **To the rigour critics:** the empirical literature is thin but positive — Ittner, Larcker & Randall 2003 in financial services found BSC-style measurement associated with better decision making and performance (§6.3) — and the practitioner generations (3rd-generation, Public Sector Scorecard, Nine Steps) added the structure the 1992 article lacked (§9.2).
- **The fair summary:** the BSC is best understood as a *hypothesis-management device* — it makes strategy testable, cascadable and reviewable. Its critics are right that the hypotheses are unproven; its defenders are right that unproven-and-tested beats unproven-and-hidden.

---


## 9. The Evolution

### 9.1 Verification: The Canonical Timeline

The evolution is **verified against the Wikipedia treatment of the BSC's history and the primary texts** consulted this edition:

| Year | Artifact | What changed | Verification |
|---|---|---|---|
| 1987 | Analog Devices "corporate scorecard" (Art Schneiderman) | Antecedent design, similar to what became "first-generation" BSC | Wikipedia treatment (flagged — secondary source) |
| 1990–91 | Kaplan-led research study with Nolan-Norton; 12 companies | The empirical base; the 1991 Nolan Norton Institute report precedes the article | Verified (1992 article text + [it_strategy_guide.md](it_strategy_guide.md) §8.2) |
| **1992** | HBR article, "The Balanced Scorecard—Measures That Drive Performance" | The four perspectives; measurement system | **Verified** (HBR archive + sibling §3.6) |
| 1993 | HBR article, "Putting the Balanced Scorecard to Work" | Four characteristics; "more than a measurement system" | Verified (1996 reprint's Further Reading) |
| **1996** | HBR article, "Using the BSC as a Strategic Management System" + the 1996 book *The Balanced Scorecard* | From measurement system → **strategic management system**; four management processes | **Verified** (reprint R0707M full text this edition) |
| **2000** | HBR article, "Having Trouble with Your Strategy? Then Map It" + book *The Strategy-Focused Organization* | **Strategy maps**; five principles of the strategy-focused organization | **Verified** (HBR archive + Wikipedia treatment) |
| 2004 | Book, *Strategy Maps: Converting Intangible Assets into Tangible Outcomes* | Full map methodology; value propositions; the intangible-asset conversion thesis | Standard attribution (flagged) |
| 2006 | Book, *Alignment: Using the Balanced Scorecard to Create Corporate Synergies* | Enterprise-level scorecards; the enterprise value proposition | Verified (2007 reprint author bio + HBS product page) |
| 2008 | Book, *The Execution Premium* | The closed-loop management system (strategy → operations) | Wikipedia treatment (verified name/date; book itself not consulted — flagged) |
| 2019 | "Intelligent Design of Inclusive Growth Strategies" (Kaplan & Norton, HBR) | Societal/inclusive-growth outcomes folded into the scorecard logic | Wikipedia treatment (verified existence; article not consulted — flagged) |

### 9.2 Generations and the Practitioner Re-inventions (verified names, via Wikipedia + BSI)

- **First generation:** the 1992 design — four perspectives, measures chosen from strategy (the Kaplan–Norton original; Schneiderman's 1987 design "similar to what is now recognised as a 'First Generation' balanced scorecard design").
- **Second generation:** added the **strategy map / strategic linkage model** and cause-and-effect (2000), making the strategy–measure link explicit.
- **Third generation:** designs that start from a "destination statement" and an outcome-driven logic rather than the four perspectives (Lawrie & Cobbold 2004; verified via Wikipedia). The **Public Sector Scorecard** and the UN's **Results Based Management** are named successors for mission-driven organisations (verified names, via Wikipedia).
- **The BSI's Nine Steps to Success (consulted this edition):** a full strategic-planning methodology that keeps the four-perspective core but renames learning & growth to **"Organizational Capacity"** (human capital, tools and technology, infrastructure, governance) — "Learning and growth takes place throughout the whole organization and during the execution of strategy, not just in one perspective." Practitioner evolution, flagged as such.
- **The software era (flagged):** the 2GC survey literature (2020, reported via Wikipedia) notes most scorecards are still reported via ad hoc means (email, office software); dedicated BSC software (automated report production and multi-scorecard coordination) is used mainly in complex multi-level organisations. Specific vendor claims were **not** re-verified this edition.

### 9.3 The ESG and Integrated-Reporting Era (verified where indicated)

The BSC's evolution now runs parallel to — and increasingly merges with — the sustainability and integrated-reporting movement:

- **Integrated reporting (<IR>) (verified, Wikipedia treatment):** the International Integrated Reporting Council's framework defines an integrated report as "a concise communication about how an organization's strategy, governance, performance and prospects lead to the creation of value over the short, medium and long term." The movement's institutional arc is verified: the 2009 Prince of Wales convening of investors, standard-setters and UN representatives; the IIRC's *International Integrated Reporting Framework* (published December 2013 — standard attribution, flagged; the framework section itself verified via the Wikipedia page); South Africa's JSE/King III as first-mover adopter; **June 2021** IIRC + SASB → **Value Reporting Foundation**; **November 2021** IFRS Foundation announcement consolidating the VRF and CDSB into the new **International Sustainability Standards Board (ISSB)** by **June 2022** (all verified via the Wikipedia treatment).
- **Why <IR> is the BSC's natural successor-language:** it is the same *idea* — one report that connects financial and non-financial value creation — moved from the management dashboard to the external annual report. The BSC's four perspectives are an internal <IR>; <IR> is the BSC externalised.
- **ESG into the scorecard (flagged where applicable):** the practice literature extends the map with environmental and social objectives (a "sustainability BSC" — an attribution to Figge, Hahn, Schaltegger & Wagner 2002 in *Business Strategy and the Environment* is standard but was **not** re-verified this edition; flagged). Kaplan & Norton's own 2019 inclusive-growth article folds societal outcomes into the framework rather than adding a fifth perspective.
- **The through-line:** 1992 (measures) → 1996 (management system) → 2000 (maps) → 2006 (alignment) → 2008 (execution) → 2010s–2020s (ESG, integrated reporting, inclusive growth). The framework has survived by absorbing each era's new outcome category without abandoning the core move: *strategy, expressed as a balanced set of cause-and-effect measures.*

### 9.4 The Evolution Table

| Era | What changed | Signature artifact | Status in this guide |
|---|---|---|---|
| Pre-1990 | Corporate scorecards at Analog Devices; *tableau de bord*; GE's 1950s measurement work | Antecedent designs | Flagged (secondary sources) |
| 1992–95 | Measurement system; four perspectives; the 12-company study | HBR Jan–Feb 1992 | Verified |
| 1996–99 | Strategic management system; four management processes; the 1996 book | HBR Jan–Feb 1996 (reprint R0707M) | Verified |
| 2000–08 | Strategy maps; strategy-focused organisation; alignment; execution premium | HBR Sep–Oct 2000; 2000/2004/2006/2008 books | Verified / flagged per item |
| 2009–now | ESG, integrated reporting, inclusive growth; practitioner generations (3rd-gen, Public Sector, Nine Steps) | IIRC <IR> framework 2013; VRF 2021; ISSB 2022 | Verified / flagged per item |

### 9.5 The Software and Tooling Landscape (flagged)

- **The BSC is, by definition, low-tech:** "typically no more than about 20 measures … easily reported manually (on paper, or using simple office software)" (verified via the Wikipedia treatment, citing Adams, Neely & Kennerley 2007). Most scorecards in the 2GC survey are still reported via ad hoc means — email, phone, office suites.
- **Dedicated BSC software** automates report production, multi-scorecard coordination and cascaded roll-ups — used mainly where several scorecards must interlock (the 2GC survey's finding, reported via Wikipedia).
- **The modern estate** has absorbed the BSC into the broader performance-management stack: strategy-execution platforms, dashboard/BI tools (the cockpit metaphor made literal — see "dashboard" in §12), and the EPM/CPM suites. Specific vendor names and market shares were **not** re-verified this edition — treat any vendor claim as unverified.
- **The honest rule** (cross-ref [it_strategy_guide.md](it_strategy_guide.md) §8.4): the tool is the *reporting* layer; the *discipline* — traceable measures, tested causality, closed review loops — is the scorecard. Software cannot save a scorecard whose measures do not trace to the map.

---

## 10. The Worked Example: A Cymbal Bank Scorecard

### 10.1 The Scenario (fictional composite — the familiar context)

**"Meridian Bank Asia"** is a fictional composite in the Cymbal Bank mould — a universal/corporate-and-investment bank with regional headquarters in Singapore: corporate banking (loans, trade finance, cash management), global markets (FX, rates, structured products), and a growing transaction-banking franchise — the business lines documented in this repo's banking cluster ([../banking/capital_markets_architecture_guide.md](../banking/capital_markets_architecture_guide.md), [../banking/universal_banking_model_guide.md](../banking/universal_banking_model_guide.md)). It is a *unification*-flavoured bank: one client view, shared platforms, heavy regulation, and a board that wants both growth and cost discipline. The scenario mirrors the sibling worked examples on purpose — [strategic_management_guide.md](strategic_management_guide.md) §4.1.1 (regional bank) and [it_strategy_guide.md](it_strategy_guide.md) §8/§10 (the Cymbal Bank IT transformation) — so the three guides can be read as one case at three levels: strategy, IT, and performance measurement.

**The strategy statement (one sentence, 1996-style):** *"To be the region's most trusted corporate and investment bank — superior service to targeted corporate and institutional clients, powered by digital operations and the deepest client franchise in the region."*

Following §3.5, the executives first build the strategy map (12–18 objectives, four layers), then derive the scorecard. The causal spine: **L&G (client-facing digital skills, risk talent) → Internal process (STP onboarding, automated credit decisions, early-warning risk) → Customer (client NPS, wallet share, digital adoption) → Financial (RoTE, cost-to-income, income growth).**

### 10.2 The Four-Perspective Design

Each row is one line of the scorecard: **objective (from the map) → measure → target → initiative.** The measures deliberately mix **lagging** (outcomes: RoTE, NPS, cost-to-income) and **leading** (drivers: decision time, STP rate, skills coverage) — the §4 discipline.

| Perspective | Objective (map) | Measure | Target (illustrative) | Initiative | Type |
|---|---|---|---|---|---|
| **Financial** | Grow profitable client revenue | Net banking income growth (C&I + markets + transaction banking) | +6% YoY, with NBI/client +3% | Client-segment pricing & cross-sell programme | Lagging |
| Financial | Optimise the cost base | Cost-to-income ratio | < 55% by year 3 | Branch/back-office digitisation; platform consolidation | Lagging |
| Financial | Deliver shareholder returns | RoTE (return on tangible equity) | > 12% | Capital allocation review (risk-weighted) | Lagging |
| **Customer** | Deepen client loyalty | Client NPS (corporate & institutional) | +15 pts over 2 years | RM retraining + relationship playbooks | Lagging |
| Customer | Grow wallet share in target segments | Share of wallet in SME / mid-cap corporates | +3 pts | Dedicated segment advisory units | Lagging |
| Customer | Win on digital experience | Digital adoption (% of transactions via digital channels) | 70% by year 3 | Client portal v3.0; e-banking API onboarding | Leading (driver of NPS & cost) |
| **Internal process** | Accelerate credit decisions | Loan decision time (straight-through, non-complex) | < 48 hours | Automated credit scoring; workflow engine | Leading |
| Internal process | Strengthen risk management | NPL ratio; early-warning coverage | NPL < 2.5%; 100% coverage of watchlist | Early-warning system; stress-testing cadence | Lagging (NPL) / Leading (coverage) |
| Internal process | Raise operational efficiency | STP rate (trade finance + payments) | 85% by year 2 | Workflow automation; exception-queue reduction | Leading |
| Internal process | Keep the bank safe and compliant | Regulatory findings; audit items closed on time | Zero material findings; 90% on-time closure | Control & compliance automation | Lagging |
| **Learning & growth** | Build digital capability | % of client-facing staff digitally certified | 60% by year 2 | Digital academy (e.g. data, API, AI) | Leading |
| L&G | Deepen risk and product talent | Succession coverage of key roles; attrition of HiPos | 90% coverage; < 5% attrition | HiPo programme; retention for critical risk roles | Leading |
| L&G | Modernise the platform | Architecture obsolescence index (% systems on supported versions); release frequency | 100% supported; monthly releases | Platform modernisation portfolio | Leading |
| L&G | Foster an innovation culture | Ideas per employee per year; % ideas piloted | 2 ideas/employee/yr; 10% piloted | Innovation lab; hackathons | Leading |

(Cross-ref [strategic_management_guide.md](strategic_management_guide.md) §4.1.1 for the sibling version of this table and [it_strategy_guide.md](it_strategy_guide.md) §8.3/§10 "Step 7 — Value" for the IT-unit version with 3-year targets.)

### 10.3 The Strategy Map, Read Back (cause-and-effect, §3)

- **The root is L&G:** the digital academy (certification), the platform modernisation and the HiPo bench are the *causes* — without them the process layer cannot move.
- **Process converts capability into service:** automated credit scoring and workflow automation *produce* the 48-hour decision; the early-warning system *produces* the NPL outcome. STP is the efficiency engine behind the cost-to-income target.
- **Customer converts service into revenue:** NPS and digital adoption are the *mechanism* by which process excellence becomes wallet share — the value proposition ("most trusted… superior service") lives at this layer.
- **Financial validates the chain:** RoTE, income growth and cost-to-income are the *end state* — and if the drivers all improve but financials don't, the 1996 strategic-learning machinery says: the theory of the business is wrong, review it (double-loop), don't just re-forecast.
- **The lagging/leading mix (§4) is visible row by row:** the scorecard has six lagging, seven leading, and two mixed rows — a rear-view mirror plus a steering wheel.

### 10.4 Cascading the Meridian Scorecard (Tier sketch, §5)

- **Tier 1 — Bank:** the table above.
- **Tier 2 — Business lines:** Global Markets (RoTE by desk, VaR limit adherence, e-trading adoption, algos STP); Trade Finance (STP rate, onboarding time, NPS of trade clients); IT (unit cost per transaction, availability, time-to-market, skills coverage — the [it_strategy_guide.md](it_strategy_guide.md) §8.3 table); Singapore branch/coverage network (cost per client, digital adoption in catchment).
- **Tier 3 — Teams/individuals:** RMs (portfolio NPS, cross-sell per client, coverage calls); credit analysts (decision time, quality of early-warning flags); engineers (release frequency, change-failure rate). Personal scorecards tie to performance and compensation, per the 1996 "communicating and linking" process.

### 10.5 The Lessons (what the worked example exists to show)

1. **The scorecard is the strategy, measured.** Every line traces to the one-sentence strategy and to a strategy-map objective; nothing is decorative. If Meridian cannot trace a measure, it drops it (§4.4).
2. **Balance is the point.** The board sees RoTE *and* NPS *and* NPL *and* skills coverage on one page — the 1992 suboptimization guard at work.
3. **Causality is a hypothesis, not a fact.** Meridian's chain (skills → STP → NPS → RoTE) is exactly what Norreklit (§8.2) says must be tested — so the review cadence (quarterly, per §6.1 process 4) checks the *links*, not just the numbers.
4. **Gaming is designed against.** Targets are few, traceable and balanced; no single metric dominates compensation; the personal scorecard mirrors the enterprise chain rather than a bonus-only subset — the Jensen (§8.3) antidote.
5. **Cascading is translation, not copying.** Each tier restates the objectives in its own measures (§5.2) — the branch network does not report "RoTE", it reports the drivers it can actually move.
6. **The 30-month lesson.** Meridian's board should expect to iterate the four management processes two or three times before the system stabilises (1996, National Insurance) — a scorecard is adopted, then *institutionalised*.

### 10.6 The Review Cadence and Governance (the 1996 processes, made concrete)

The scorecard only works if something *happens* on a rhythm. A workable Meridian cadence:

| Rhythm | What is reviewed | Who | The 1996 process behind it |
|---|---|---|---|
| **Monthly** | Operational drivers: STP, decision time, digital adoption, availability — the leading indicators | Unit heads + COO | Business planning; feedback on execution |
| **Quarterly** | The full scorecard: outcomes vs targets, and the *links* — "did STP improvement show up in NPS?" | Exco + unit heads | Feedback and learning; the causal model tested |
| **Annual** | Strategy reaffirmation or revision: targets recalibrated, initiatives re-prioritised, map updated | Board + Exco | Translating the vision, restarted; double-loop learning |
| **On trigger** | Any disconfirming evidence (NPS drops while drivers improve; NPL rises with no warning fired) | Exco immediately | The 1996 "early warning" logic — treat it as a theory failure, not a numbers miss |

Governance rules: one owner per measure (attribution is a governance act — [it_strategy_guide.md](it_strategy_guide.md) §8.4); every initiative has a named sponsor and a benefit-realisation review at 90 days (cross-ref [business_case_development_guide.md](business_case_development_guide.md)); and the quarterly review asks the Norreklit question out loud — *"which causal link are we assuming, and what would disconfirm it?"*

### 10.7 What Success Looks Like (and What It Does Not)

- **Success:** a one-page scorecard that a new hire can read and say "I know what we are trying to do, and I know which line is mine" (the 1996 "strategy in a bottle" ideal); a quarterly meeting where someone *changes the strategy* on the evidence (double-loop learning); a budget that visibly follows the targets.
- **Not success:** a beautifully rendered dashboard nobody reviews (the "reporting exercise nobody reads" failure — [it_strategy_guide.md](it_strategy_guide.md) §8.4); a scorecard where every measure is green because targets were set to be safe; a cascade where Tier 3 employees cannot name their Tier-1 objective.

### 10.8 The Board Pack — Presenting the One-Pager

The scorecard reaches the board as exactly what it claims to be: one page. A Meridian board pack:

1. **Page 1 — the scorecard:** the §10.2 table, colour-coded (green/amber/red against target), with the map printed small beside it so the board can read the *links*, not just the colours.
2. **Page 2 — the variance story:** the three or four lines that moved most since last quarter, each with a driver explanation ("STP rose 6 pts because of workflow automation; NPS has not yet followed — the causal link is under test, expected lag two quarters").
3. **Page 3 — the strategic-learning note:** what the evidence says about the theory of the business, and the one decision the board must take (reaffirm, recalibrate targets, or revise strategy) — the 1996 feedback-and-learning process, formatted for a governance audience.
4. **The rule of the pack:** if the board pack needs more than three pages, the scorecard has too many measures or the strategy is not yet one page.

---

## 11. The Summary: The Strategy on One Page

The BSC's promise, in the 1992 article's own framing, is to give top managers "a fast but comprehensive view of the business" — the strategy on one page. Here is that page:

| Perspective | The question (1992, verbatim) | What it measures | The causal role |
|---|---|---|---|
| **Financial** | "How do we look to shareholders?" | RoTE, income growth, cost-to-income, cash flow | The outcome — the strategy's end state |
| **Customer** | "How do customers see us?" | NPS, retention, market share, digital adoption | Converts process excellence into revenue |
| **Internal process** | "What must we excel at?" | Cycle time, STP, quality, risk outcomes | Produces the customer value proposition |
| **Learning & growth** | "Can we continue to improve and create value?" | Skills, systems, climate, innovation | The root — the cause of everything above |

**The final word — the strategy on one page.** The Balanced Scorecard survived thirty years of fads, critics and management-tool surveys because it answered a permanent problem with a permanent form: strategy is abstract, but *measurement is concrete*. The four questions — shareholders, customers, excellence, improvement — force any strategy into a shape that can be agreed, communicated, cascaded, budgeted and reviewed. The strategy map shows the causal logic; the measures make it testable; the cascade makes it everyone's; the review cycle makes it live. Whatever the era — industrial, information, ESG — the one-page discipline is the same: *if you cannot put your strategy on one page of linked measures, you do not have a strategy you can execute.* That is the BSC's enduring thesis, and it is why this guide's title is not a metaphor: the Balanced Scorecard *is* the strategy on one page.

### 11.1 The Five-Question Self-Test (any scorecard, any unit)

1. **Do my measures answer the four questions?** Shareholders, customers, excellence, improvement — if a measure answers none of them, drop it (§2.8).
2. **Does my map have a causal spine?** Can I draw the arrows from learning & growth up to financial, and defend each one? (§3.2, §8.2)
3. **Do I mix lagging and leading?** Outcomes alone are a rear-view mirror; drivers alone are activity (§4.1).
4. **Does every line have an owner, a target and an initiative?** A measure without a target is decoration; a target without an initiative is a wish (§4.4).
5. **Does the review change anything?** If the quarterly meeting never revises strategy or recalibrates targets, the scorecard is a report, not a management system (§6.1, process 4).

---

## 12. The Glossary

| Term | Definition |
|---|---|
| **Balanced scorecard (BSC)** | A strategy performance-management tool: a focused set of financial and non-financial measures, organised into perspectives, compared against targets, with a portfolio of initiatives — the strategy expressed as one report. Origin: Kaplan & Norton, HBR, Jan–Feb 1992 (verified) |
| **Kaplan** | Robert S. Kaplan — Harvard Business School accounting professor (Arthur Lowes Dickinson Professor at the time of the 1992 article; later Marvin Bower Professor of Leadership Development emeritus); co-creator of the BSC with Norton |
| **Norton** | David P. Norton — president of Nolan, Norton & Co. (IT consulting); co-creator of the BSC; cofounder with Kaplan of the Balanced Scorecard Collaborative / Palladium |
| **Financial perspective** | "How do we look to shareholders?" — the top layer for for-profit organisations: ROI/ROE, growth, cash flow, cost efficiency (1992 question, verbatim; verified) |
| **Customer perspective** | "How do customers see us?" — market share, retention, satisfaction, the time/quality/performance-and-service/cost categories (1992, verbatim; verified) |
| **Internal process perspective** | "What must we excel at?" — the processes that produce the customer value proposition: cycle time, quality, efficiency, risk outcomes (1992, verbatim; verified) |
| **Learning and growth perspective** | "Can we continue to improve and create value?" — skills, systems, culture and innovation capacity; the root of the causal chain (1992, verbatim; verified) |
| **Strategy map** | A graphic of strategic objectives (ovals) across the four perspective layers, connected by cause-and-effect arrows — the strategy drawn. Introduced in Kaplan & Norton, "Having Trouble with Your Strategy? Then Map It," HBR, Sep–Oct 2000 (verified) |
| **Cause-and-effect** | The BSC's core hypothesis: L&G → internal process → customer → financial; the links that make the scorecard a theory of the business rather than a metric list (1992 implicit; 1996 explicit; contested by Norreklit 2000) |
| **Lagging indicator** | An outcome measure that reports results after the fact — profit, NPS, cost-to-income. "Financial measures that tell the results of actions already taken" (1992, verified) |
| **Leading indicator** | A driver measure that predicts future outcomes — decision time, STP rate, skills coverage. Operational measures "that are the drivers of future financial performance" (1992, verified) |
| **Cascading** | Translating the enterprise scorecard down through business units (Tier 2) to teams and individuals (Tier 3) — the 1996 "communicating and linking" process; formalised by the Balanced Scorecard Institute as Tier 1/2/3 (verified) |
| **Alignment** | The state where every unit's and individual's objectives trace to the enterprise strategy; the subject of Kaplan & Norton, *Alignment* (2006, verified) |
| **KPI** | Key Performance Indicator — a quantitative measure tied to an objective, compared against a target; the BSC's unit of measurement (BSI definition, consulted this edition) |
| **Measure** | The metric assigned to a strategic objective on the scorecard (lagging or leading, financial or non-financial) |
| **Target** | The desired value of a measure by a date — the ambition that turns a measure into a commitment |
| **Initiative** | The action programme/project that moves a measure toward its target — the "how" behind the "what" |
| **Jensen** | Michael C. Jensen — economist (Harvard Business School); author of the 2001 critique ("Value maximisation, stakeholder theory, and the corporate objective function," *European Financial Management* 7(3): 297–318) that the BSC provides no overall score and invites gaming (verified citation) |
| **Norreklit** | Hanne Norreklit — accounting scholar (Aarhus/Århus); author of the 2000 critique ("The balance on the balanced scorecard," *Management Accounting Research* 11(1): 65–88) that the BSC's cause-and-effect assumptions are unvalidated (verified citation) |
| **ESG** | Environmental, Social and Governance — the sustainability outcome categories that the 2010s–2020s era folds into strategy and reporting; the BSC's fourth-decade extension (see §9.3) |
| **Integrated reporting (<IR>)** | The IIRC framework for one report connecting strategy, governance, performance and prospects to value creation over time — the BSC's idea externalised to the annual report; IIRC → Value Reporting Foundation (2021) → ISSB (2022) (verified via Wikipedia treatment) |
| **Dashboard** | A visual report of measures against targets; the BSC's cockpit metaphor made literal ("the dials and indicators in an airplane cockpit," 1992) — related to the French *tableau de bord* tradition (flagged) |
| **Scorecard** | The BSC report itself: objectives, measures, targets and initiatives across the four perspectives — "the strategy on one page" |

---

## Appendix: The Blank Scorecard Template

Use this to start any scorecard — unit, IT, branch, or enterprise. The discipline: fill the map first (§3.5), then this table; every row must trace to a map objective.

| Perspective | Strategic objective (map) | Measure | Target (by when) | Initiative (sponsor) | Type (lagging/leading) |
|---|---|---|---|---|---|
| **Financial** | | | | | |
| **Customer** | | | | | |
| **Internal process** | | | | | |
| **Learning & growth** | | | | | |

Rules of the template: no more than ~20 measures in total (1992's "handful"); at least one leading driver under every lagging outcome; every target has a date; every initiative has a named owner; and the whole page must fit on one screen — the strategy on one page.

---

## 13. Verification Notes and Sources

### What was verified this edition (primary or authoritative sources)

- **1992 origin:** HBR archive page ("From the Magazine (January–February 1992)"; reprint 92105) and full reprint text of "The Balanced Scorecard—Measures That Drive Performance" (steinbeis-bi.de PDF): the four questions, the cockpit metaphor, the 12-company research project, "what you measure is what you get", the four customer concerns (time, quality, performance and service, cost), the ECI example, the "kill another tree program" quote, the senior-manager-involvement finding. Doubly verified with [mba_body_of_knowledge_guide.md](mba_body_of_knowledge_guide.md) §3.6 (HBR 70(1):71–79).
- **1996 strategic management system:** full reprint R0707M ("Using the Balanced Scorecard as a Strategic Management System," reissued July–August 2007; originally January–February 1996): four management processes, Metro Bank, National Insurance (30 months), the "rubber meets the sky" quote, strategic learning / double-loop learning, the >100-organisation claim.
- **2000 strategy maps:** HBR archive page confirming "Having Trouble with Your Strategy? Then Map It," September–October 2000.
- **1993 article:** "Putting the Balanced Scorecard to Work," HBR September–October 1993, with its four characteristics (via the 1996 reprint's Further Reading annotations).
- **Norreklit 2000** and **Jensen 2001** citations: via the Wikipedia treatment's reference list (full bibliographic details, DOIs — see §8.1).
- **BSI practice canon:** the Four Perspectives page, the Strategy Map page, and the Cascading page (balancedscorecard.org, consulted this edition): Tier 1/2/3, organizational-capacity rename, 12–18 objectives, public-sector inversion.
- **Integrated reporting:** Wikipedia treatment — IIRC framework definition, 2009 origins, 2021 VRF, 2022 ISSB consolidation.
- **Sibling cross-refs:** [mba_body_of_knowledge_guide.md](mba_body_of_knowledge_guide.md) §3.6 (origin), [it_strategy_guide.md](it_strategy_guide.md) §8 (IT scorecard + leading/lagging), [strategic_management_guide.md](strategic_management_guide.md) §4.1 (banking scorecard), [business_case_development_guide.md](business_case_development_guide.md) (benefit realisation).

### What is flagged (not re-verified this edition)

- Analog Devices 1987 / Schneiderman's "corporate scorecard" coinage and the *tableau de bord* lineage (secondary source: Wikipedia treatment).
- The 2004 *Strategy Maps* book and 2008 *The Execution Premium* book details (standard attributions; the books themselves not consulted).
- The "sustainability BSC" attribution (Figge, Hahn, Schaltegger & Wagner 2002).
- Adoption statistics: the 2GC 2020 survey figures (88% strategy-implementation use, 63% operational, 17% personal) are reported via the Wikipedia treatment and not re-verified against 2GC's own report; Bain Management Tools historical adoption figures were **not** retrievable this edition (web search backend intermittently unavailable).
- BSC software vendor claims (see §9.2).
- The web_search backend was intermittently unavailable during this edition; where it failed, verification was completed via direct extraction of primary/authoritative URLs (HBR, BSI, Wikipedia, the 1992 reprint PDF).

### Sources consulted this edition

- **HBR archive pages:** "The Balanced Scorecard—Measures That Drive Performance" (January–February 1992, hbr.org/1992/01); "Having Trouble with Your Strategy? Then Map It" (September–October 2000, hbr.org/2000/09); "Using the Balanced Scorecard as a Strategic Management System" (reprint, July–August 2007, hbr.org/2007/07).
- **Full reprints read this edition:** the 1992 article reprint (reprint 92105, steinbeis-bi.de PDF) — the four questions, ECI case, cockpit metaphor, measure-proliferation warning; the 1996 article reprint (R0707M, Microsoft-hosted PDF) — the four management processes, Metro Bank, National Insurance, strategic learning.
- **Balanced Scorecard Institute (balancedscorecard.org, consulted this edition):** "The Four Perspectives of the Balanced Scorecard"; "About Strategy Mapping"; "Cascading Creating Alignment"; the *Nine Steps to Success* references.
- **Wikipedia (consulted this edition):** "Balanced scorecard" (history, criticism, software, 2GC 2020 survey figures, Norreklit/Jensen/Ittner citations); "Integrated reporting" (IIRC history, VRF 2021, ISSB 2022).
- **Sibling repo guides (cross-referenced, not re-researched):** [mba_body_of_knowledge_guide.md](mba_body_of_knowledge_guide.md) §3.6; [it_strategy_guide.md](it_strategy_guide.md) §8; [strategic_management_guide.md](strategic_management_guide.md) §4.1–4.3; [business_case_development_guide.md](business_case_development_guide.md); [organizational_behavior_guide.md](organizational_behavior_guide.md); [management_consulting_skills_guide.md](management_consulting_skills_guide.md); [mckinsey_approach_guide.md](mckinsey_approach_guide.md); the banking and technology clusters listed in the header.

### The one-sentence provenance statement

Every un-flagged claim in this guide traces to one of: the 1992 article reprint, the 1996 article reprint, HBR's archive pages, the Balanced Scorecard Institute's current web pages, the Wikipedia treatment of the BSC and of integrated reporting, or the sibling guides listed in §13's first block — and where a claim could not be traced, it is flagged.

---

*End of guide — the strategy on one page.*
