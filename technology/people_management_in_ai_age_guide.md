# People Management in the AI Age: Extinction or Evolution?

**Author:** Jack Liu Shurui  
**Context:** Solution Architect, Crédit Agricole CIB, Singapore  
**Series:** Management & AI/LLM Intersection  
**Last Updated:** July 2026

---

## Table of Contents

1. [The Premise: When AI Can Do Everything a Manager Does](#1-the-premise-when-ai-can-do-everything-a-manager-does)
2. [Two Contrasting Viewpoints](#2-two-contrasting-viewpoints)
3. [What AI Can Do Well in Management](#3-what-ai-can-do-well-in-management)
4. [What AI Can't Do (At All) in Management](#4-what-ai-cant-do-at-all-in-management)
5. [The Changing Role of the People Manager](#5-the-changing-role-of-the-people-manager)
6. [The AI Threat by Management Level](#6-the-ai-threat-by-management-level)
7. [The Future of Management in the AI Age](#7-the-future-of-management-in-the-ai-age)
8. [The Banking Context](#8-the-banking-context)
9. [Conclusions](#9-conclusions)

---

## 1. The Premise: When AI Can Do Everything a Manager Does

Consider the following list. Every single item below is something that, as of mid-2026, AI tools can do — not theoretically, but practically, at scale, in production environments:

- **Write code** that ships to production (GitHub Copilot, Claude Code, Cursor)
- **Review pull requests** for bugs, style violations, security issues, and architectural concerns
- **Manage project backlogs** — prioritize tickets, suggest sprint compositions, flag stale items
- **Analyze team performance data** — velocity trends, cycle time distributions, deployment frequency
- **Write performance reviews** — first drafts incorporating peer feedback, commit history, and project contributions
- **Schedule meetings** — find optimal times across time zones, book rooms, send calendar invites
- **Take meeting notes** — transcribe, summarise, extract action items, track decisions
- **Answer team questions** — about codebases, processes, architectural decisions, company policies
- **Give feedback** — on code quality, documentation clarity, presentation effectiveness, communication style
- **Generate status reports** — daily standup summaries, weekly digests, monthly stakeholder updates
- **Monitor team health** — track sentiment via Slack/Teams message analysis, flag burnout risk, identify disengagement patterns
- **Optimise team composition** — recommend skill adjacencies, identify knowledge silos, suggest cross-training opportunities

**The uncomfortable question:** If AI can do all of this — and do it faster, cheaper, and often more consistently than a human manager — do we still need people managers?

This is not a rhetorical question. It is being asked in boardrooms, HR departments, and engineering leadership meetings across every industry. The answers range from "middle management as we know it is dead" to "management has never been more important." Both perspectives contain truth. The real answer, as this piece will argue, is more nuanced: **management is not dying — it is evolving, and the evolution demands a fundamentally different kind of manager.**

---

## 2. Two Contrasting Viewpoints

### Viewpoint A: "AI Will Replace Middle Management"

This argument, popularised in tech circles and VC thought leadership, goes as follows:

> "AI can coordinate work, analyse performance, write reports, and give feedback. What exactly is the middle manager adding that justifies their compensation? The future is a small team of ICs guided by AI, reporting to a single senior leader or tech lead."

**The case for this view:**

- **Information relay is automated.** Much of middle management's historical value was serving as an information conduit between executives and teams. AI can now aggregate, summarise, and distribute information more effectively than any human.
- **Coordination overhead collapses.** AI tools can track dependencies, flag blockers, optimise resource allocation, and manage cross-team handoffs without a human orchestrator. Companies like GitLab and Basecamp have already demonstrated that async, highly-documented workflows reduce management needs.
- **Performance management can be data-driven.** If you have objective metrics (velocity, quality, impact), AI can evaluate performance more fairly than a human manager subject to recency bias, halo effects, and personal relationships.
- **The "Amazon case."** Amazon's data-driven management model — where metrics and mechanisms drive decision-making more than manager intuition — could be seen as a precursor to an AI-managed organisation. Extrapolate: replace the manager who reads the metrics with an AI that acts on them directly.

**Notable proponents:** Tech CEOs who have publicly stated they could run their companies with far fewer managers; VC firms investing in "AI-first" organisational structures where AI agents manage workflows; and researchers studying the impact of AI on knowledge work productivity.

### Viewpoint B: "Management Is More Important Than Ever in the AI Age"

This counterargument, often from experienced leaders and organisational psychologists, contends:

> "As AI automates the operational and analytical aspects of management, the human elements — trust, inspiration, psychological safety, career development, ethical judgment — become the *only* differentiator. If anything, we need *more* and *better* managers."

**The case for this view:**

- **Automation increases the value of connection.** When AI handles the transactional aspects of management, the human manager's time is freed to focus on what AI cannot do. This is not a reduction — it is an *amplification* of the human role.
- **The AI transition is traumatic.** Teams are anxious about AI replacing their jobs. This anxiety requires human leadership — reassurance, vision, honest conversations about the future, and personalised support for each team member's journey.
- **AI creates new complexities.** Managing a hybrid team of humans and AI agents introduces challenges around accountability, decision authority, role clarity, and trust that require sophisticated human judgment.
- **The "Noah's Ark" principle.** In the flood of automation, the most valuable managers are those who can build the ark — help their teams adapt, reskill, and find new value. This is fundamentally a human coaching and change management challenge.

**Notable proponents:** Organisational psychologists studying the human impact of AI; forward-thinking CHROs at companies investing heavily in manager development; and experienced engineering leaders who have lived through multiple technology transformations.

### Reconciliation: Both Are True

The resolution is not "AI replaces managers" versus "AI doesn't replace managers." The real answer is:

> **"AI replaces *some* managers and makes the rest far more valuable."**

Specifically:
- AI replaces managers whose primary value is **information relay, task coordination, and operational oversight**.
- AI makes *more* valuable managers who focus on **people development, culture building, strategic thinking, and ethical judgment**.

The middle management layer is not being eliminated — it is being *remoulded*. The number of "coordinators" will shrink. The number of "coaches" should grow.

---

## 3. What AI Can Do Well in Management

Before we can understand the human manager's future, we must honestly assess where AI excels. This is not a small list.

### Data Analysis and Insights

| Capability | Description | Current State (2026) |
|---|---|---|
| **Velocity/throughput analysis** | Track team output over time, identify trends, flag anomalies | Excellent — AI can detect patterns invisible to humans |
| **Cycle time analysis** | Measure time from ticket creation to delivery, segment by work type | Excellent — automated, continuous, granular |
| **Deployment frequency tracking** | Monitor CI/CD pipeline metrics, correlate with team velocity | Excellent — native to DevOps tooling |
| **Bottleneck identification** | Pinpoint where work gets stuck — waiting for review, blocked by dependencies, unclear requirements | Very good — AI can trace across multiple systems |
| **Delivery date prediction** | Forecast completion dates based on historical velocity, uncertainty estimation | Good — better than human intuition for routine work |
| **Team performance correlation** | Identify what predicts high performance (e.g., pairing frequency, PR size, meeting load) | Very good — AI can find correlations humans miss |

**What this means:** Many of the data-crunching activities that managers used to spend hours on — manual spreadsheet analysis, building dashboards, writing performance reports — can now be done by AI in seconds. The human manager's job shifts from *gathering and presenting data* to *interpreting and acting on it*.

### Documentation and Communication

| Capability | Description | Current State |
|---|---|---|
| **Performance reviews** | Draft reviews incorporating peer feedback, commit data, project contributions, and growth trajectory | Very good — especially when review criteria are well-defined |
| **Status reports** | Generate weekly/monthly stakeholder updates from project management data | Excellent — can be fully automated |
| **Meeting summaries** | Transcribe, summarise, extract action items and decisions | Excellent — often better than human notes |
| **Decision logs** | Track and document architectural and process decisions with rationale | Very good — when integrated with comms tools |
| **Project documentation** | Generate and maintain living documentation from code, PRs, and discussions | Good — requires human validation of accuracy |

**What this means:** The administrative burden of management documentation — arguably the least enjoyable part of the job for most managers — is increasingly automated. This is arguably *good* for management, freeing time for higher-value human activities.

### Scheduling and Coordination

| Capability | Description | Current State |
|---|---|---|
| **Meeting optimisation** | Find optimal times across participants, time zones, preferences | Excellent — standard in calendar tools |
| **Calendar management** | Schedule, reschedule, cancel, manage conflicts | Excellent — AI schedulers are mature |
| **Cross-timezone coordination** | Handle complex timezone maths for distributed teams | Excellent |
| **Resource allocation** | Suggest who should work on what based on skills, workload, and priorities | Good — improving rapidly |

**What this means:** The "meeting scheduler" and "calendar wrangler" aspects of management are fully automatable. Managers should never manually schedule another meeting.

### Code Review and Technical Oversight

| Capability | Description | Current State |
|---|---|---|
| **Automated PR review** | Style enforcement, documentation checks, test coverage validation | Excellent — standard in CI pipelines |
| **Bug detection** | Identify potential bugs, logic errors, edge cases | Very good — improving with each model generation |
| **Security scanning** | Detect vulnerabilities, hardcoded secrets, dependency risks | Excellent — automated scanning is table stakes |
| **AI pair programming** | Real-time code suggestions, refactoring assistance, test generation | Very good — Copilot, Claude, Cursor, etc. |
| **Architecture review** | Flag architectural concerns, suggest design improvements | Good — best for well-known patterns, weaker for novel designs |

**What this means:** First-line tech leads whose primary value is code review and technical oversight are significantly augmented. Their role shifts from "catches all the bugs" to "makes judgment calls on what the AI flags."

### Project Management

| Capability | Description | Current State |
|---|---|---|
| **Backlog prioritisation** | Score tickets by value, urgency, dependencies, team capacity | Very good — when criteria are well-defined |
| **Sprint planning** | Suggest sprint compositions balancing capacity, priorities, dependencies | Good — requires human input for fuzzy priorities |
| **Dependency tracking** | Identify and visualise cross-team and cross-system dependencies | Excellent — automated dependency graphs |
| **Risk identification** | Flag delivery risks based on scope changes, velocity drops, blocker patterns | Very good — predictive risk scoring |

**What this means:** Routine project management is increasingly AI-driven. The manager's role shifts from *managing the backlog* to *managing the conversation around the backlog* — the human negotiation between competing priorities, stakeholder expectations, and team capacity.

### Basic Coaching and Support

| Capability | Description | Current State |
|---|---|---|
| **Technical question answering** | Answer questions about codebases, processes, best practices | Very good — especially with organisational knowledge base |
| **Documentation provision** | Surface relevant docs, guides, examples on demand | Excellent |
| **Best practice suggestions** | Recommend patterns, approaches, tools based on context | Good — improving |
| **Process guidance** | Explain team workflows, escalation paths, decision frameworks | Very good |

**What this means:** Routine coaching — "how do I do X?" "what's the team's approach to Y?" — can be handled by AI. This frees the human manager to focus on *developmental* coaching: helping team members grow their careers, build judgment, and navigate complex interpersonal situations.

---

## 4. What AI Can't Do (At All) in Management

This section is critical. If the previous section showed that AI can do a lot of what managers *do*, this section shows that AI cannot do most of what makes managers *effective*.

### Building Trust

AI can simulate empathy — it can say "I understand this must be frustrating" in a convincingly warm tone. But it cannot build genuine trusting relationships.

**Why it matters for management:** Trust is the foundation of every management interaction. When a team member brings you a problem, they need to trust that you will:
- Maintain confidentiality
- Act in their interest
- Give honest, constructive feedback
- Have their back in difficult situations
- Be consistent in your values and responses

AI can *simulate* the behaviours of a trustworthy person. But trust requires a relationship built over time, through consistent actions, through demonstrated commitment. No AI can genuinely *care* about a team member's career — and that caring, over time, is what trust is made of.

**Concrete example:** A senior engineer tells their manager they're considering leaving the company. A human manager can have a genuine conversation about what's missing, explore the real reasons (which the engineer may not have fully articulated to themselves), advocate internally for changes, and follow through — building deeper trust in the process. An AI assistant can suggest retention strategies based on industry data, but it cannot *care* whether the engineer stays or goes.

### Creating Psychological Safety

Psychological safety — the belief that you can take risks, admit mistakes, and speak up without being punished — is one of the most researched predictors of team performance. It is also profoundly human.

**Why it matters for management:** Google's Project Aristotle found that psychological safety was the #1 predictor of team effectiveness. Creating this environment requires:
- Modelling vulnerability ("I made a mistake, here's what I learned")
- Responding non-defensively to challenges
- Protecting people who speak up
- Consistently signalling that learning is valued over blame

AI cannot model genuine vulnerability — it can say the words, but it cannot *feel* the risk or *demonstrate* the courage that makes vulnerability meaningful to humans.

**Concrete example:** During a post-mortem, a team member admits they introduced a bug that caused a production incident. The manager's response in that moment — is it blame-focused or learning-focused? — determines whether that team member ever admits a mistake again. AI can generate a "blameless post-mortem" template, but it cannot read the room, calibrate tone, or demonstrate the authentic non-defensiveness that builds safety over time.

### Reading the Room

Emotionally intelligent managers can walk into a room and within seconds sense tension, morale issues, or unspoken dynamics. This is a complex, embodied skill that AI fundamentally lacks.

**Why it matters for management:** Much of a manager's most important work happens in the space *between* what people say:
- Two team members avoid eye contact during a standup — there's a conflict
- An engineer who usually contributes is silent in a design review — they're disengaged or feeling unheard
- The energy in a team meeting feels flat — morale is down
- A joke falls flat — there's tension about a recent decision

AI, even with perfect sentiment analysis on written communication, cannot perceive the embodied, contextual, real-time emotional dynamics of a group of humans in a room (or even on a video call).

**Concrete example:** A manager notices during a sprint retro that the team seems unusually quiet. Team members give one-word answers and avoid looking at each other. The manager suspects there's an unspoken issue with a recent process change. They pause the retro, name what they're observing ("I'm sensing there might be something we're not saying"), and create space for the real conversation to emerge. AI cannot perceive that quiet nor sense its meaning.

### Career Coaching

Understanding someone's aspirations, fears, motivations, and creating a personalised growth path is one of the most deeply human management activities.

**Why it matters for management:** Career coaching requires:
- Understanding the individual's unique combination of skills, values, personality, and life circumstances
- Seeing potential the person doesn't see in themselves
- Having honest conversations about strengths and growth areas
- Creating opportunities tailored to the individual
- Advocating for the person in ways that align with their specific career goals

AI can generate career development plans based on industry data and common career paths. But it cannot *know* an individual the way a thoughtful manager does through months or years of one-on-ones, observing them in different situations, and building a relationship.

**Concrete example:** A junior engineer is technically strong but struggles with confidence. Their manager notices they avoid presenting their work and stay quiet in design reviews. Rather than just assigning communication training (which AI might recommend), the manager works with them over months — starting with small, safe presentations, providing detailed positive feedback, gradually increasing scope. The manager is betting time, attention, and emotional energy on this person's growth. AI cannot make that bet.

### Conflict Resolution

Mediating between team members with nuanced understanding of history, personalities, and organisational politics is deeply human.

**Why it matters for management:** Conflict resolution requires:
- Understanding the full context — not just what was said, but the history, personalities, power dynamics, and organisational pressures
- Building trust with both parties so they're willing to be vulnerable
- Navigating emotionally charged conversations without escalating them
- Finding solutions that respect everyone's legitimate interests
- Maintaining relationships with all parties after the conflict

AI can suggest conflict resolution frameworks (non-violent communication, interest-based negotiation, etc.) but cannot *mediate* a live conflict between two humans.

**Concrete example:** Two senior engineers disagree on a technical approach. The disagreement has escalated to personal attacks in Slack. The manager listens to each person separately, understands the underlying concerns (one feels their past contributions weren't valued, the other feels micromanaged), then facilitates a conversation where both feel heard. The manager's knowledge of each person's history, motivations, and communication style is essential to resolving this — and only possible because of a long-term relationship.

### Inspiration and Vision Articulation

Articulating a compelling vision that makes people *want* to follow is a uniquely human capability.

**Why it matters for management:** Inspiration requires:
- Genuine conviction and passion that others can *feel*
- The ability to connect day-to-day work to a larger purpose
- Storytelling that resonates emotionally, not just logically
- Authenticity — people can tell when you're reading from a script
- The courage to set direction in the face of uncertainty

AI can generate beautifully worded mission statements and vision documents. But inspiration is transmitted through human connection — through voice, body language, eye contact, and the authentic emotion that makes people believe.

**Concrete example:** A team is working on a complex migration that feels like thankless infrastructure work. The manager helps them see why it matters — how this migration will enable the team to ship features ten times faster next year, how it will reduce the cognitive load that's been burning everyone out, how it positions the team for bigger, more interesting work. The manager shares their own excitement about the future and their commitment to making the migration as painless as possible. AI can write the business case. It cannot make anyone *care*.

### Context Giving

Helping people understand *why* their work matters in the broader organisational and human context.

**Why it matters for management:** Humans are meaning-seeking creatures. When we understand why our work matters, we are more engaged, more creative, and more resilient. Giving context requires:
- Understanding the broader business strategy and translating it to team-level meaning
- Connecting individual tasks to customer impact, company goals, and personal values
- Making the invisible visible — showing how a refactoring project enables future speed
- Adapting the story to what resonates with each team member

AI can generate context — it can explain how a feature connects to quarterly OKRs. But it cannot *feel* the importance of the work or communicate it with the authenticity that makes it land.

**Concrete example:** A data engineer is frustrated with a tedious data quality project that feels beneath their skills. The manager explains how this data quality work directly impacts the bank's regulatory reporting — and that getting this right prevents fines, protects the bank's reputation, and ultimately keeps people in their jobs. The manager shares a specific story about a past incident where poor data quality caused real harm. The engineer leaves the conversation with a renewed sense of purpose. AI can explain the business case. It cannot tell the story that makes it *personal*.

### Difficult Conversations

Firing someone, delivering bad news, addressing sensitive performance issues — these are the hardest parts of management and the most resistant to AI replacement.

**Why it matters for management:** Difficult conversations require:
- Empathy that is *felt*, not performed — the other person needs to know you care
- The courage to deliver honest, potentially painful feedback
- Navigating strong emotions (tears, anger, denial) with grace
- Making judgment calls about timing, setting, and framing
- Living with the emotional weight of the conversation afterward

AI can generate scripts for difficult conversations. But the conversation itself — the real-time navigation of another person's pain, fear, or anger — requires a human who can be present, compassionate, and honest in a way that an AI fundamentally cannot.

**Concrete example:** A manager needs to tell a talented engineer that they're being laid off due to restructuring. The manager knows this engineer just bought a house and has a baby on the way. The conversation requires the manager to be present with that pain, to answer questions honestly, to offer genuine support, and to help the engineer leave with dignity. No AI can deliver that conversation.

### Sponsorship

Advocating for someone's career, opening doors, making introductions, betting your reputation on them.

**Why it matters for management:** Sponsorship is different from mentorship — it's an active investment of social capital. It requires:
- Believing in someone's potential enough to stake your reputation
- Using your network and influence to create opportunities
- Making introductions that matter
- Advocating for promotions, stretch assignments, and visibility
- Being willing to say "I'll personally vouch for this person"

AI cannot build social capital, invest it in others, or feel the risk that comes with sponsorship. These actions depend on relationships, trust, and the willingness to put yourself on the line for someone else.

**Concrete example:** A manager nominates a high-potential engineer for a critical project that's visible to senior leadership. The manager personally briefs the project lead on the engineer's strengths, coaches the engineer on how to succeed in the role, and checks in weekly to ensure they're set up for success. When the engineer faces challenges, the manager advocates for them behind the scenes. This active sponsorship — which takes time, energy, and reputational risk — is deeply human.

### Cultural Stewardship

Defining and maintaining team values, norms, and identity is a leadership function that AI cannot replicate.

**Why it matters for management:** Culture is not a document or a set of rules. It is the *lived experience* of a group of people — how decisions are made, how conflicts are handled, what behaviours are rewarded, what is taboo. Maintaining culture requires:
- Modelling the values you want to see
- Catching small deviations before they become norms
- Having conversations about "how we do things here"
- Celebrating behaviours that reinforce the culture
- Confronting behaviours that undermine it
- Adapting culture as the team evolves

AI can write a "team culture document." It cannot *live* the culture or *shape* it through daily interactions.

**Concrete example:** A team has a culture of "disagree and commit" — people are expected to challenge decisions during discussion but fully support the outcome once a decision is made. A new team member consistently undermines decisions after they're made. The manager has a one-on-one conversation about the norm, explaining why it matters for team effectiveness and trust. Over time, through consistent modelling and reinforcement, the culture is maintained. AI cannot model or shape culture.

### Moral Judgment

Navigating ethical gray areas, making decisions that balance multiple stakeholders with competing values.

**Why it matters for management:** Management is full of decisions with no clear right answer:
- Balancing individual team member needs against team performance
- Deciding when to protect someone from consequences vs. when to let them face accountability
- Choosing between short-term delivery pressure and long-term team health
- Navigating competing stakeholder interests
- Making judgments about fairness, equity, and inclusion

AI can apply rules consistently. But management requires *wisdom* — the ability to make good judgments in ambiguous, value-laden situations where the rules don't give a clear answer.

**Concrete example:** Two high-performing team members have a personality conflict that's starting to affect team morale. Both are valuable and the manager doesn't want to lose either. There's no policy that tells the manager what to do. They need to understand both perspectives, find a solution that respects both individuals while protecting the team, and make a judgment call about what trade-offs are acceptable. This requires moral reasoning, empathy, and contextual wisdom — not rule-following.

### Creativity Facilitation

Helping teams unlock creative solutions through conversation, provocation, and psychological safety.

**Why it matters for management:** Innovation doesn't happen in a vacuum. It happens in environments where people feel safe to propose wild ideas, challenge assumptions, and build on each other's thinking. Facilitating this requires:
- Creating space for divergent thinking
- Asking provocative questions that challenge assumptions
- Building on ideas surfaced by others
- Managing group dynamics to ensure all voices are heard
- Knowing when to push and when to pull back
- Making creative collaboration feel safe and energising

AI can generate creative ideas. It cannot *facilitate* a group of humans to generate creative ideas together — to read the room, adjust the energy, draw out the quiet person, and create the spark that leads to collective insight.

**Concrete example:** During a brainstorming session, the team is stuck generating incremental improvements. The manager notices one junior team member has an idea but is hesitating to share it. They explicitly invite the person to speak. The idea is unconventional but interesting — the manager doesn't dismiss it but instead builds on it, showing that unconventional thinking is valued. This unlocks a cascade of more creative ideas from other team members. The manager's facilitation — the noticing, the invitation, the positive reinforcement — is what created the breakthrough. AI cannot notice, invite, or build in that human way.

---

## 5. The Changing Role of the People Manager

### The New Model: AI as Co-Manager

The most productive framing is not "AI replaces managers" but "AI becomes a co-manager." The human manager and the AI manager each handle what they do best:

| Function | Handled By |
|---|---|
| Data gathering and analysis | AI |
| Report generation and documentation | AI |
| Scheduling and coordination | AI |
| Monitoring and alerting | AI |
| First drafts and templates | AI |
| Trend identification and pattern recognition | AI |
| **---** | **---** |
| Interpretation and decision-making | Human |
| Relationship building and trust | Human |
| Coaching and development | Human |
| Strategic direction | Human |
| Culture building | Human |
| Ethical judgment | Human |
| Inspiration and vision | Human |
| Conflict resolution | Human |

The manager's role shifts from **"director"** (telling people what to do, tracking what they did, reporting up) to **"coach + context-giver + culture-builder"** (developing people, connecting their work to purpose, maintaining the team's health and values).

### New Skills for AI-Age Managers

#### 1. AI Literacy

Managers need to understand what AI can and cannot do — not at the level of an ML engineer, but at the level of an informed consumer and evaluator.

**What this means in practice:**
- Knowing when to trust AI-generated analysis vs. when to question it
- Understanding the limitations and failure modes of current AI tools
- Being able to evaluate AI output critically (is this analysis sound? is this recommendation biased?)
- Recognising where AI adds value vs. where it introduces risk
- Staying current as AI capabilities evolve rapidly

#### 2. Data Interpretation

As AI generates more analytics and insights, the manager's role shifts from *producing* data to *interpreting* it and turning it into action.

**What this means in practice:**
- Reading AI-generated team health dashboards and deciding what to address
- Interpreting velocity trends in context (is this a real decline or a seasonal variation?)
- Turning AI-identified patterns into coaching moments
- Making resource allocation decisions based on AI recommendations, but applying human judgment about context
- Communicating data insights to the team in a way that motivates rather than demoralises

#### 3. Prompt Engineering for Management

Crafting effective prompts for AI management assistants is becoming a practical skill.

**What this means in practice:**
- Structuring prompts to get useful performance review drafts
- Asking AI to analyse specific team dynamics while avoiding bias
- Using AI to generate options for difficult situations, then applying human judgment
- Iterating on prompts to improve the quality and relevance of AI outputs
- Knowing how to chain AI outputs into coherent management workflows

#### 4. Ethical AI Use

Managers must ensure AI is used ethically in management contexts — especially in performance evaluation, hiring, and promotion.

**What this means in practice:**
- Auditing AI-generated performance reviews for bias (race, gender, tenure effects)
- Ensuring AI hiring tools don't discriminate against protected groups
- Being transparent with the team about how AI is used in management decisions
- Maintaining human accountability for AI-assisted decisions
- Understanding regulatory requirements for AI in HR (EU AI Act, local employment laws)
- Establishing "human in the loop" checkpoints for consequential decisions

#### 5. Hybrid Team Leadership

Teams increasingly include both humans and AI agents. Managing this hybrid system is a new leadership challenge.

**What this means in practice:**
- Understanding the "capabilities and limitations" of each AI tool on your team
- Deciding which tasks go to humans, which to AI, and which to human+AI collaboration
- Managing team dynamics when humans feel threatened by AI tools
- Ensuring AI agents don't undermine human motivation or agency
- Creating workflows where humans and AI complement each other
- Handling accountability when an AI agent makes a mistake — who owns it?

#### 6. Change Management

AI disruption is ongoing and accelerating. Managers need to help their teams continuously adapt.

**What this means in practice:**
- Helping team members understand how AI changes their roles
- Identifying reskilling needs and creating development paths
- Managing anxiety and resistance around AI adoption
- Communicating the vision for how the team will evolve
- Creating a culture of continuous learning and adaptation
- Modelling the behaviours desired in the AI-augmented workplace

### What Managers Should Stop Doing

| Activity | Why | Replacement |
|---|---|---|
| Spending hours on status reports | AI can generate them from project management tools | Review AI-generated reports in minutes |
| Manual data analysis in spreadsheets | AI can analyse faster and find patterns humans miss | Ask AI for insights, apply judgment |
| Writing first drafts of performance reviews | AI can draft based on data and feedback | Review, personalise, and add human insight |
| Micromanaging task allocation | AI can optimise assignments based on skills and capacity | Focus on development and motivation |
| Scheduling coordination | AI schedulers handle this better | Verify, don't coordinate |

### What Managers Should Double Down On

| Activity | Why It Matters |
|---|---|
| **1:1s** | The foundation of trust, coaching, and connection — even more important when other interactions are AI-mediated |
| **Career conversations** | AI can't know someone's aspirations or create personalised growth paths |
| **Team building** | Human connection is the antidote to AI-induced isolation and anxiety |
| **Creating psychological safety** | The #1 predictor of team effectiveness — and only humans can build it |
| **Strategic thinking** | AI provides data, humans provide direction and judgment |
| **Stakeholder management** | Navigating organisational dynamics is deeply human |
| **Coaching and mentoring** | Personalised, relationship-based development is irreplaceable |
| **Recognising and celebrating wins** | Human appreciation is fundamentally different from AI-generated praise |
| **Navigating organisational politics** | Protecting the team, building coalitions, managing up |
| **Building resilience and adaptability** | Helping teams thrive through continuous change |
| **Fostering innovation and creativity** | Creating the conditions for human creative collaboration |

### The AI-Augmented Manager Workflow

A typical week for an AI-augmented manager might look like:

**AI handles (asynchronously):**
- Aggregates team velocity data and generates weekly performance summary
- Analyses sentiment from team communication tools and flags potential issues
- Drafts weekly status report for stakeholders
- Reviews open pull requests and flags high-risk changes
- Identifies backlog items that are stale or blocked
- Suggests optimal meeting times for upcoming one-on-ones and team events
- Monitors deployment frequency, cycle time, and failure rate
- Generates first draft of performance review for the next review cycle

**Human manager focuses on (in team time):**
- **Monday morning:** Reviews AI-generated weekly summary with critical eye. Notes a velocity decline that the AI attributed to "increased complexity" — decides to investigate whether it's actually a morale or process issue.
- **Monday 1:1s:** Has five one-on-ones. In one, a team member shares they're feeling burnt out. The manager commits to adjusting their workload and follows up with specific changes.
- **Tuesday:** Facilitates a design review session. Notices the team's best designer is quiet — draws them into the conversation, which leads to a breakthrough.
- **Wednesday:** Mediates a conflict between two engineers who disagree on an architectural approach. Listens to both, understands the underlying concerns, and helps them find a third path.
- **Thursday:** Reviews AI-generated stakeholder report, adds context the AI couldn't know (a political dynamic with another team), and presents it to leadership.
- **Friday:** Holds a team retrospective. Uses AI-generated data on sprint performance to start the conversation, then spends the real time on what the data doesn't show — team morale, collaboration quality, and improvement ideas.

---

## 6. The AI Threat by Management Level

Not all management levels are equally affected. The impact varies significantly.

### First-Line Managers (Tech Leads, Team Leads)

**Vulnerability: HIGH**

First-line managers are the most vulnerable to AI disruption. Their traditional value proposition — technical oversight combined with task coordination — is precisely where AI excels.

**What's being automated:**
- Code review (AI does first pass, human handles edge cases)
- Task assignment and coordination (AI optimises based on skills and capacity)
- Technical guidance on well-known patterns (AI provides answers instantly)
- Sprint planning and backlog grooming (AI suggests compositions)
- Status tracking and reporting (AI generates automatically)

**The new role:**
The first-line manager shifts from "senior IC who also manages" to "people coach and culture carrier." Their technical authority now comes from *judgment* — knowing which AI suggestions to accept and which to question — rather than from having more technical knowledge than the team.

**At risk:** First-line managers whose primary value is being the best coder on the team and tracking everyone's progress.

**Safe:** First-line managers who excel at developing people, creating team cohesion, and translating business strategy into meaningful team context.

### Middle Managers (Engineering Managers, Directors)

**Vulnerability: MODERATE**

Middle managers face a more nuanced disruption. AI handles their reporting and analytics load but cannot replace their coaching, alignment, and organisational integration functions.

**What's being automated:**
- Report generation for higher management
- Cross-team coordination (dependency tracking, status aggregation)
- Performance data analysis and presentation
- Resource planning scenarios and projections
- Meeting logistics and calendar management

**The new role:**
Middle managers increasingly focus on coaching other managers, maintaining organisational culture across teams, translating strategy into execution, and ensuring career development across a broader organisation. They become less "the person who reports on what's happening" and more "the person who makes what's happening better."

**At risk:** Middle managers whose primary function is information relay between executives and teams.

**Safe:** Middle managers who add value through strategic thinking, organisational design, talent development, and culture building.

### Senior Management (VPs, CTOs)

**Vulnerability: LOW**

Senior management is the least vulnerable to AI disruption. Strategy, vision, organisational design, stakeholder management, and culture shaping are deeply human activities.

**What AI provides:**
- Better data for decision-making (analytics, scenario modelling, market intelligence)
- Faster analysis of strategic options
- Automated monitoring of organisational health metrics
- Drafting and iteration support for strategic communications

**What remains human:**
- Setting strategic direction in the face of uncertainty
- Making judgment calls that balance multiple stakeholders with competing values
- Building trust with executive peers, board members, and external partners
- Shaping organisational culture through visible behaviour and decisions
- Making high-stakes ethical judgments
- Inspiring the organisation through vision and authenticity

**At risk:** Very few senior roles, but some strategic planning functions may be augmented.

**Safe:** Most senior leader roles — AI augments their data and analysis but cannot replace their judgment, leadership, or relationship-building.

### The Big Picture: AI Flattens Management, Not Eliminates It

The overall effect is **flattening**, not elimination:

- **Reduces** the need for managers whose primary value is information relay and task coordination (thinner middle layers)
- **Increases** the need for managers who develop people, build culture, and provide context (more coaching-focused roles)
- **Transforms** the manager's job from transaction-based to relationship-based
- **Raises the bar** for what "good management" means — the easily automated parts are automated, leaving only the hard parts

A useful analogy: AI does for management what the calculator did for mathematics and what the spreadsheet did for accounting. It automates the drudgery and elevates the role to focus on higher-value thinking — but only for those who adapt.

---

## 7. The Future of Management in the AI Age

### New Management Models

Several models are emerging for how management works when AI handles the operational load:

#### Manager as Coach

Already trending before AI, this model accelerates dramatically. When AI handles task coordination, reporting, and basic coaching, the manager's job becomes purely developmental — helping people grow, solve their own problems, and reach their potential.

**Core activities:** Active listening, powerful questioning, providing feedback, creating stretch opportunities, career mapping, skill development.

**Best for:** Teams where the work is creative, complex, or knowledge-intensive.

#### Manager as Context-Giver

The manager focuses on providing the *why* while AI handles the *how*. The manager's primary value is connecting the team's work to business strategy, customer impact, and personal meaning.

**Core activities:** Translating strategy into team context, sharing customer stories, explaining the rationale behind priorities, connecting individual work to organisational goals, maintaining alignment with broader company direction.

**Best for:** Large organisations where teams can become disconnected from business impact.

#### Manager as Culture-Builder

In a world where AI coordinates the work, the human manager's job is to maintain the team's health, values, and identity. Culture becomes the primary differentiator between high-performing and mediocre teams.

**Core activities:** Defining and modelling team values, maintaining psychological safety, celebrating wins and learning from failures, managing team rituals and traditions, onboarding new members into the team's culture.

**Best for:** Distributed teams where culture is harder to maintain organically.

#### Manager as AI Orchestrator

A new model where the manager's primary skill is managing the human+AI system — deciding which tasks go to humans, which to AI, and how they collaborate.

**Core activities:** Selecting and configuring AI tools for the team, designing human+AI workflows, training the team on AI tools, monitoring AI output quality, adjusting allocation as AI capabilities evolve.

**Best for:** Teams that are early adopters of AI tools or in industries where AI capability is advancing rapidly.

### Skills That Become More Valuable

| Skill | Why It Matters More |
|---|---|
| **Emotional intelligence** | The quality that differentiates human managers from AI — reading emotions, building relationships, navigating conflict |
| **Strategic thinking** | AI provides data and options, humans choose direction and make judgment calls |
| **Creativity** | AI can generate but can't facilitate human creative collaboration |
| **Ethical judgment** | As AI takes on more management tasks, oversight of bias, fairness, and accountability becomes critical |
| **Coaching ability** | The core of the new manager role — helping people grow |
| **Change management** | Helping teams navigate continuous AI-driven disruption |
| **Cross-functional influence** | Building alignment across teams without authority — a deeply human skill |
| **Storytelling** | Making meaning, providing context, inspiring — AI can write words but can't tell stories that resonate |
| **Systems thinking** | Understanding how AI, humans, processes, and technology interact in complex organisational systems |

### Skills That Become Less Valuable

| Skill | Why It Matters Less |
|---|---|
| **Task tracking** | AI monitors progress automatically |
| **Status reporting** | AI generates reports from project management data |
| **Manual data analysis** | AI analyses faster and finds patterns humans miss |
| **Schedule optimisation** | AI handles scheduling logistics |
| **Information relay** | AI distributes information more efficiently |
| **Process enforcement** | AI can enforce processes consistently |
| **Remembering details** | AI remembers everything perfectly |

### Team Structures Evolving

The traditional team structure — one manager, 5-10 direct reports — evolves toward new configurations:

**Smaller human teams, augmented by AI agents:**
- Teams of 3-5 humans supported by multiple AI agents
- Each AI agent has specific capabilities (code review, analysis, documentation, project management)
- The manager coordinates both humans and AI agents

**Managers managing different "species":**
- Humans and AI agents have different "motivations," "capabilities," and "limitations"
- The manager needs different strategies for motivating people vs. configuring agents
- Humans need coaching, development, and psychological safety
- AI agents need proper configuration, prompt engineering, and output validation

**Task allocation becomes strategic:**
- The manager decides which tasks go to humans (creative, ambiguous, relationship-based)
- Which go to AI (repetitive, analytical, well-defined)
- Which benefit from human+AI collaboration (complex problem-solving, creative ideation)
- These decisions change as AI capabilities evolve

### Organisational Implications

**Flatter structures:**
- AI reduces coordination overhead, enabling wider spans of control
- Fewer middle managers needed for information relay
- More emphasis on team autonomy with AI handling coordination

**New roles emerging:**
- **AI-Agent Manager:** Manages the fleet of AI agents supporting a team — configuring, monitoring, updating
- **Human-AI Interaction Designer:** Designs workflows for effective human-AI collaboration
- **AI Training Lead:** Curates and updates the knowledge base that AI management assistants draw from
- **Organisational AI Ethicist:** Ensures AI tools are used fairly and ethically across management processes

**Measurement and accountability shifts:**
- Teams measured on outcomes and impact, not activity and hours
- Managers evaluated on team health and development, not reporting accuracy and coordination efficiency
- Accountability for AI-augmented decisions remains human (AI is a tool, not a decision-maker)

---

## 8. The Banking Context

### Why Bank Management Is Different

Management in banking — and particularly in a regulated tier-1 bank like Crédit Agricole CIB — operates under constraints that make the AI transition distinct:

**Regulatory requirements mean human judgment remains essential.**

#### Key Regulatory Frameworks

**SR 11-7 (Federal Reserve — Model Risk Management):**
- Any AI tool used in management decisions that could affect the bank constitutes a "model"
- Requires independent validation, documentation, and ongoing monitoring
- Human oversight is mandatory — AI recommendations must be reviewed by qualified humans
- The bank's leadership is accountable for model risk, not the AI

**MAS FEAT Principles (Monetary Authority of Singapore):**
- Fairness, Ethics, Accountability, Transparency in AI use
- AI-driven decisions (including management and HR decisions) must be explainable
- Human accountability is non-delegable — you cannot blame an AI for a bad decision
- Regular auditing for bias and fairness is required

**EU AI Act (Effective 2025-2026):**
- Classifies certain management-related AI uses as "high-risk" (hiring, performance evaluation, promotion decisions)
- High-risk AI systems require conformity assessments, human oversight, and transparency
- Fines for non-compliance can be significant (up to 7% of global annual turnover)
- Human review of high-risk AI decisions is mandatory

### Implications for Banking Managers

| Requirement | Implication |
|---|---|
| **Regulatory compliance** | AI management tools must be validated, documented, and monitored — managers must understand and oversee this process |
| **Human accountability** | Managers remain accountable for AI-assisted decisions — they can't delegate responsibility to AI |
| **Bias management** | AI-driven performance evaluation and hiring must be audited for fairness and compliance |
| **Transparency** | Team members must know when and how AI is used in management decisions affecting them |
| **Risk management** | AI tools introduce new risks that must be managed within the bank's risk framework |

### The Irreplaceable Human in Banking Management

Several aspects of banking management are particularly resistant to AI replacement:

**Trust with regulators:**
Regulators want to speak with *people* — not read AI-generated compliance reports. The relationship between a bank's management and its regulators is built on personal trust, credibility, and demonstrated judgment. AI cannot show up to a regulatory meeting, look a regulator in the eye, and answer a tough question about model governance with the right combination of candour and confidence.

**Understanding complex regulatory context:**
Regulatory requirements are nuanced, evolving, and often subject to interpretation. An AI might correctly apply a regulation but miss the *intent* or the broader regulatory environment. Human managers bring contextual understanding that is critical for navigating regulatory complexity.

**Risk/reward judgment calls:**
Banking is fundamentally about balancing risk and reward. AI can model scenarios and calculate probabilities, but the *decision* — how much risk is acceptable for a given return, in the current market and regulatory environment, given the bank's risk appetite and strategic priorities — requires human judgment.

**Building team resilience through change:**
Banking is undergoing massive technological transformation, and the pace is only accelerating. Managers who can help their teams navigate this change — through coaching, honest communication, and genuine support — are invaluable. AI can suggest change management frameworks but cannot be present for the difficult conversations.

**Maintaining ethical standards in AI adoption:**
Banks have a particular responsibility to use AI ethically — both because of regulatory requirements and because of the trust society places in financial institutions. Banking managers play a critical role in ensuring that AI adoption doesn't cut corners on ethics, fairness, or compliance. This stewardship role is deeply human.

**Concrete example:** A banking team is implementing an AI tool to assist with performance reviews. The tool might introduce biases related to the types of projects that get visibility, the communication styles it evaluates positively, or the way it weighs quantitative vs. qualitative contributions. A banking manager needs to:
- Understand the regulatory implications (MAS FEAT, SR 11-7 if applicable)
- Oversee bias testing of the AI tool
- Make judgment calls about how to weight AI-generated inputs vs. human assessment
- Maintain transparency with the team about how AI is being used
- Carry ultimate accountability for the fairness and accuracy of performance evaluations

This is not a task that can be delegated to an AI — it requires human judgment, ethical reasoning, and accountability.

---

## 9. Conclusions

### Management Is Not Dying — It Is Evolving

The narrative that "AI will replace managers" is too simple. What is actually happening is a fundamental transformation of what management means:

- **The operational coordinator role is being automated.** The manager as task tracker, report writer, schedule wrangler, and information relay is being replaced by AI. This is not a loss — it is liberation from the least valuable parts of the job.
- **The human leader role is becoming more important than ever.** As AI handles the transactional, the human manager is freed to focus on what only humans can do: build trust, develop people, create culture, provide context, make ethical judgments, and inspire.

### AI Will Not Replace Managers Who Develop People, Build Culture, and Provide Context

The managers who are safe — and more valuable than ever — are those who:

- Invest deeply in one-on-ones and career development
- Create psychological safety where teams can take risks and learn from failure
- Build genuine trust through consistent, authentic behaviour
- Navigate organisational politics to protect and advocate for their teams
- Help people understand why their work matters
- Maintain team culture and values
- Make wise ethical judgments in ambiguous situations
- Facilitate creative collaboration
- Have the courage for difficult conversations

### AI Will Replace Managers Whose Only Value Is Tracking and Reporting

The managers who are at risk are those who:

- Spend most of their time writing status reports and analysing spreadsheets
- Add little value beyond "the person who knows what everyone is doing"
- Cannot coach or develop their team members
- Lack emotional intelligence and relationship-building skills
- Are not trusted by their teams for meaningful career conversations
- Cannot navigate organisational complexity or strategic ambiguity

### The Best Managers in the AI Age Will Be Those Who...

**1. Embrace AI tools to free up time for human-centric management.**

The worst thing a manager can do in the AI age is ignore the tools. The second worst thing is to let AI do the human work. The right approach is to use AI aggressively for what it's good at — data analysis, documentation, scheduling, monitoring — so you have more time for what only you can do.

**2. Focus on what only humans can do.**

Build trust. Inspire. Coach. Navigate politics. Make ethical judgments. Have difficult conversations. Create psychological safety. Facilitate creative collaboration. Celebrate wins. Provide context. These are not just "nice to have" — they are the *core* of management in the AI age.

**3. Continuously learn and adapt as AI capabilities evolve.**

AI is not static. What AI can't do today, it may be able to do next year. Managers need to stay current — not at the level of ML engineers, but with a working understanding of what AI can and cannot do for their teams.

**4. Help their teams navigate the same transition.**

The most valuable managers in the AI age will be those who help their team members — who are also facing AI disruption — navigate their own transitions. This means:
- Honest conversations about how AI affects their roles
- Identifying new skills and creating development paths
- Building resilience and adaptability
- Maintaining optimism and purpose in the face of change
- Creating an environment where learning AI tools is safe and supported

### Final Thought

> **"AI will not replace managers. But managers who use AI will replace those who don't."**

This is not a quip — it is a prediction grounded in the economic logic of the AI transition. The operational aspects of management are being commoditised by AI. The relational aspects are becoming more scarce and more valuable. Managers who embrace this shift — who use AI to eliminate drudgery and invest the freed time in human connection — will be more valuable than ever. Managers who ignore AI, or who cling to the transactional aspects of the job, will find themselves increasingly redundant.

The question is not whether management survives the AI age. The question is whether *you* — as a manager or aspiring manager — will evolve with it.

---

## References and Further Reading

- Google re:Work — Project Aristotle: psychological safety research
- Adam Grant — *Think Again* (on coaching and feedback culture)
- Kim Scott — *Radical Candor* (on caring personally while challenging directly)
- Michael Bungay Stanier — *The Coaching Habit* (on coaching-based management)
- Patrick Lencioni — *The Five Dysfunctions of a Team* (on trust and psychological safety)
- EU AI Act (2024) — regulatory framework for high-risk AI systems
- MAS FEAT Principles — Fairness, Ethics, Accountability, Transparency guidelines
- SR 11-7 — Federal Reserve guidance on model risk management
- Harvard Business Review — various articles on AI and management (2024-2026)
- McKinsey — "The Future of Management in an AI-Driven World" (2025)
- Gartner — "Predicts 2026: AI's Impact on Management and Leadership"
- MIT Sloan Management Review — "The AI-Augmented Manager" (2025)

---

*This document is part of the Management & AI/LLM series. Complementary guides in this repository cover situational leadership (SLII), the 3D Managerial framework, managing up/down/sideways, 360° management, first-time management, and various AI/LLM technical guides.*
