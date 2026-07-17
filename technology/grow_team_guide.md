# Growing a Team: A Comprehensive Guide

> **Author:** Jack Liu Shurui
> **Role:** Solution Architect, Crédit Agricole CIB
> **Context:** Research reference — technology/ directory
> **Last updated:** July 2026

---

## Table of Contents

1. [Hiring](#1-hiring)
2. [Onboarding](#2-onboarding)
3. [Team Culture](#3-team-culture)
4. [Scaling](#4-scaling)
5. [Development](#5-development)
6. [Performance Management](#6-performance-management)
7. [Team Topology](#7-team-topology)
8. [Architect Perspective: Platform & Infrastructure Teams](#8-architect-perspective-platform--infrastructure-teams)

---

## 1. Hiring

### 1.1 Writing Effective Job Specifications

A job spec is your first pitch to a candidate. It must be precise, honest, and differentiated.

**Structure a job spec in four sections:**

- **Context, not just responsibilities.** Open with the team's mission and how this role fits into broader organisational goals. A candidate should understand *why* this role exists, not just *what* the daily tasks are. Example: "You will own the customer identity platform that serves 2M daily active users — this means designing for security, scale, and developer experience across 15 product teams."
- **Outcomes over activities.** Instead of "manage Kubernetes clusters", write "ensure 99.99% uptime for containerised workloads across three regional clusters, reducing deploy-to-production lead time from 45 minutes to under 10." Measurable outcomes attract high-agency candidates.
- **Must-have vs nice-to-have.** Be ruthless. Every nice-to-have filters out candidates who might excel. List no more than 4–5 must-haves. Clearly label everything else as "bonus" or "would be great if".
- **Culture and team norms.** Describe how the team works — async-first? pair-programming? on-call rotation? — so candidates self-select. A senior engineer who values deep focus will avoid a team that describes "fast-paced, always-on collaboration."

**Pitfalls to avoid:** overly long requirement lists (they deter women and underrepresented groups disproportionately), buzzword-dense descriptions ("rockstar ninja"), and salary ranges that are too wide or absent entirely. In Singapore's market, be explicit about EP/SPass sponsorship limitations and the expected notice period.

### 1.2 Sourcing Channels

Diversify sourcing to avoid monoculture and dependency on a single pipeline:

| Channel | Best for | Caveat |
|---------|----------|--------|
| Employee referrals | Quality, cultural insight | Can create echo chambers — cap referral % at 30% |
| LinkedIn Recruiter | Scale, passive candidates | Expensive; require strong employer brand to convert |
| GitHub / Open Source portfolios | Engineers who ship | Time-intensive to review; great for senior ICs |
| Tech-specific job boards (Stack Overflow, AngelList) | Active seekers | Smaller talent pool than LinkedIn |
| Internal mobility | Retention, institutional knowledge | Creates a backfill chain — plan for it |
| Meetups / conferences | Niche skills (e.g. Rust, secure enclaves) | Long lead time; relationship-building required |
| Talent communities / newsletters | Passive, warm leads | Takes months to build; high conversion when it works |

**Key principle:** The best hires often come from your team's professional network. Incentivise referrals with meaningful bonuses (SGD 5,000–15,000 depending on seniority), paid out in two tranches — half on start date, half after 6 months.

### 1.3 Structured Interviewing

Unstructured interviews ("tell me about yourself" + resume walkthrough) have near-zero predictive validity for job performance. Structured interviews are the single highest-impact change you can make to your hiring process.

**Components of a structured interview:**

- **Same questions, same rubric.** Every candidate for the same role answers the same core questions. Score answers on a pre-defined 1–5 scale with behavioural anchors (e.g. "3: described a specific situation with measurable outcome").
- **Competency-based questions (STAR method).** Ask for *Situation, Task, Action, Result* — and probe. "What was your specific contribution?" "How did you measure success?" "What would you do differently?" Surface-level answers are a red flag.
- **Work-sample tests.** The best predictor of job performance is a miniature version of the actual job. For a platform engineer: "Here is our Terraform state with a drift issue — diagnose and fix it." For a backend engineer: "Extend this API endpoint to handle pagination with cursor-based keys." Time-box to 90 minutes maximum.
- **Panel interviews with diverse perspectives.** Include a peer, a manager, a cross-functional stakeholder, and someone from a different background. Each panel member scores independently before discussion — this prevents halo effects and groupthink.
- **Anti-bias calibration.** Before panel discussions, have each interviewer share their scores *before* hearing others. Train interviewers on common biases: confirmation bias (seeking evidence that confirms initial impression), similarity bias (favouring candidates like yourself), and the horns/ halo effect (one strong/ weak signal colouring everything).

**Scorecard dimensions (example for a Senior Platform Engineer):**

1. Technical depth — system design, troubleshooting, automation
2. Collaboration & communication — explaining trade-offs, documentation
3. Ownership & bias for action — unprompted initiative, accountability
4. Learning velocity — how they handle unknowns and new domains
5. Alignment with team values — specific to your culture (e.g. "default to open")

### 1.4 Assessing for Culture Add vs Culture Fit

"Culture fit" is often a cover for hiring people who remind us of ourselves. Replace it with **culture add** — what new perspectives, experiences, and working styles does this person bring that the team currently lacks?

**Practical approach:**

- Define 3–5 cultural attributes your team values (e.g. "direct but respectful feedback", "write things down", "bias toward shipping"). These are non-negotiable.
- For everything else, look for complement rather than mirror. If your team is strong on execution but weak on long-term thinking, hire someone who naturally asks "what happens in Q3?".
- In the interview, ask: "Describe a team culture you thrived in. What made it work?" and "Describe a culture you struggled with. Why?" This reveals what environment the candidate needs to succeed.
- Assess during the reference check: "How did this person influence the team's dynamics? What unique perspective did they bring?"

---

## 2. Onboarding

Onboarding is the highest-leverage investment you can make in a new hire. A great onboarding experience correlates with 58% higher retention after 2 years (SHRM data).

### 2.1 30-60-90 Day Plans

Break the first three months into clear phases, each with its own goal:

**Days 1–30: Absorb**
- Goal: Understand the system, the team, and the domain.
- Deliverables: Set up dev environment; shadow onboarding buddy through a full sprint; read runbooks and architecture docs; submit at least one small PR (docs fix, test, minor bug); meet each team member in a 1:1.
- Manager check-in: Weekly 30-minute 1:1s with explicit agenda: "What's confusing? Who have you met? Any blockers?"

**Days 31–60: Contribute**
- Goal: Own a small-to-medium feature or task end-to-end.
- Deliverables: Ship at least one meaningful change independently; participate in on-call rotation (with shadow support); identify one improvement to the team's tooling or process; give a 15-minute tech talk on something they learned.
- Manager check-in: Bi-weekly 1:1s. Shift from directive to coaching: "What do you think we should do about X?"

**Days 61–90: Extend**
- Goal: Act as a reliable team member who can operate without supervision.
- Deliverables: Handle on-call incidents independently; contribute to planning and estimation; propose and lead a small improvement initiative; begin reviewing others' PRs.
- Manager check-in: Weekly 1:1s. Focus on strategic alignment: "Where do you want to go next?"

### 2.2 Buddy Systems

Every new hire needs a buddy — a peer who is not their manager. The buddy answers the questions the new hire does not want to bring to their manager: "Where is the real decision-making documented?" "Which stand-up can I skip?" "How do I get access to the staging DB?"

**Buddy responsibilities (first 4 weeks):**
- Daily 15-minute check-in for week 1, then 2–3 times per week.
- Help set up local dev environment and navigate internal tooling.
- Introduce the new hire to key people across the org.
- Flag any red flags (confusion, isolation, frustration) to the manager.

**Choosing a buddy:** Pick someone who is patient, communicative, has been on the team for at least 6 months, and is not overloaded. Rotate buddy duty so it is not a burden on a single person.

### 2.3 Documentation

A well-documented team enables self-service onboarding and reduces tribal knowledge dependency.

**Essential onboarding documentation:**

- **Environment setup guide:** Step-by-step from `git clone` to running the full test suite. Include common pitfalls and fixes. Must be tested by someone new every quarter.
- **Architecture overview:** High-level system diagram (C4 level 1-2), data flow diagrams, key technology choices and rationale.
- **Runbooks:** Incident response playbooks, common alerts and their first-line responses, deployment procedures, rollback steps.
- **Glossary:** Acronyms, team-specific terms, project names, domain concepts.
- **Who to ask for what:** A RACI-style table mapping topics (e.g. "DB schema changes", "cloud credits", "prod access") to people.

**Maintenance:** Treat docs as code — include doc updates in PRs, review stale docs every quarter, and surface outdated pages.

### 2.4 Early Wins

Nothing builds confidence like shipping. Within the first 30 days, ensure the new hire experiences the full cycle of making a change and seeing it in production.

**Design for early wins:**
- Curate a backlog of small, well-scoped tasks tagged "good first issue" with clear acceptance criteria.
- Pair with a senior team member on a deliverable that has high visibility but low risk.
- Celebrate the first deploy publicly — a team channel shout-out, a mention in stand-up.

**Warning:** Do not overload early wins. A new hire who ships five trivial fixes in week one learns nothing about the system and may feel the work is boring. Balance small wins with exposure to complex problems they observe senior engineers solving.

---

## 3. Team Culture

Culture is the set of behaviours that are tolerated, rewarded, and punished. It is not a mission statement — it is what actually happens when you are not in the room.

### 3.1 Psychological Safety

Google's Project Aristotle identified psychological safety as the #1 predictor of team effectiveness. It is the shared belief that you will not be punished or humiliated for speaking up with ideas, questions, concerns, or mistakes.

**How to build it:**

- **Model vulnerability.** As the leader, admit mistakes publicly. "I was wrong about the migration timeline — here is what I missed and what we will do differently." This gives permission for others to do the same.
- **Frame work as learning problems, not execution problems.** Instead of "why did you deploy a broken config?", ask "what in our pipeline failed to catch this, and how do we improve it?"
- **Actively solicit input.** In meetings, call on quiet voices. "Alice, you have deep experience with this pattern — what are we missing?"
- **Respond productively to bad news.** When someone surfaces a problem early, thank them. If they get punished (even subtly — a sigh, a cold tone), you train the team to hide problems.
- **Run blameless post-mortems.** The question is not "whose fault?" but "what systemic factors allowed this to happen?" Every incident is a design opportunity.

### 3.2 Feedback Culture

High-performing teams give frequent, specific, and actionable feedback. This does not happen by accident — it must be modelled, taught, and reinforced.

**Frameworks:**

- **SBI (Situation-Behaviour-Impact):** "In yesterday's sprint review (S), when you presented the latency numbers before the feature demo (B), it confused the stakeholders because they expected to see the UI first (I)."
- **Radical Candour (Scott):** Care personally while challenging directly. The goal is feedback that is both kind and clear.
- **Start-Stop-Continue:** A lightweight retrospective format for peer feedback. "Start doing X, stop doing Y, continue doing Z."

**Structural support:**
- Dedicate time in every 1:1 for upward feedback. "What could I do differently as your manager?" If you never ask, you never hear.
- Use retrospectives as a regular forum for process feedback. Make the last section of every retro about interpersonal dynamics (lightweight — "how are we communicating?")
- Pair feedback with growth. A critique without a growth path feels like a judgement.

### 3.3 Communication Norms

Explicit norms remove ambiguity about how the team communicates, especially in hybrid/remote settings.

**Recommended norms:**

- **Default to written.** Before scheduling a meeting, write a proposal. Async communication scales better and creates a searchable record. If a discussion goes beyond 2–3 messages, then schedule a call.
- **Document decisions.** Every significant decision gets a short ADR (Architecture Decision Record) or a Slack thread summary with a final conclusion pinned. This prevents repeated debates and helps new team members catch up.
- **Define response time expectations.** "Slack messages answered within 4 hours during working hours. If something is urgent, tag with @here or escalate via phone."
- **Meeting hygiene:** Publish agendas 24h in advance. Start and end on time. No meeting without a clear outcome. Use the last 5 minutes for "what did we decide and who is doing what."
- **Status updates are written, not spoken.** Use async stand-ups in Slack or a tool like Geekbot. Reserve synchronous stand-up for one specific use: surfacing blockers that need real-time discussion.

### 3.4 Remote & Hybrid

Hybrid teams face coordination asymmetry — some people are in the room, some are not. Without deliberate design, remote participants become second-class citizens.

**Practices that work:**

- **One remote, all remote.** If even one person is remote for a meeting, everyone joins from their own laptop. No "room full of people plus a speakerphone." This eliminates the asymmetry of who can read body language and side-converse.
- **Async-first culture.** Decisions are made in documents, not in meetings. Record (or at least summarise) every sync meeting for those who could not attend.
- **Over-communicate context.** Remote team members miss hallway conversations and overheard decisions. Write down the rationale behind decisions in the relevant ticket, document, or channel. Assume nothing is common knowledge.
- **Invest in digital presence.** Good cameras, good microphones, digital whiteboards (Miro, Excalidraw), and a culture of turning cameras on (with flexibility — camera-off is fine when context allows).
- **Deliberate social connection.** Remote culture does not happen naturally. Schedule virtual coffees, co-working sessions, and (critically) in-person offsites every 3–6 months for teams that are geographically distributed.
- **Output over hours.** In a remote setting, you cannot see people working. Evaluate on outcomes delivered, not time spent at a desk. This requires clear goals and trust.

---

## 4. Scaling

Scaling a team is qualitatively different from growing incrementally. The structures and habits that worked for a team of 5 break at 15.

### 4.1 When to Hire vs Contract

| Situation | Hire FTE | Contract |
|-----------|----------|----------|
| Core product work | ✅ | ❌ |
| Predictable, long-term need | ✅ | ❌ |
| Short-term spike (migration, project) | ❌ | ✅ |
| Exploratory / unproven initiative | ❌ | ✅ |
| Specialised skill needed temporarily (e.g. penetration test) | ❌ | ✅ |
| Building institutional knowledge | ✅ | ❌ |

**Guideline:** If you need the work done for 6+ months and it is core to your mission, hire FTE. If the work is time-bounded, experimental, or non-core, contract. Always ensure contractors are integrated into team rituals so they are not excluded from context.

### 4.2 Span of Control: 4–8 Direct Reports

A manager's span of control directly affects both their effectiveness and the team's experience.

- **4–5 reports** — High-touch management. Appropriate for junior-heavy teams, new managers, or high-complexity domains (e.g. security, SRE).
- **6–8 reports** — Standard range. Works when the team is experienced and the domain is stable.
- **8+ reports** — Risk zone. Unless the team is extremely senior and self-directing, the manager becomes a bottleneck for 1:1s, coaching, and career conversations. 1:1s compress to 15 minutes, skipping the deep career conversations.

**Signals your span is too wide:** You cannot remember every team member's current project. 1:1s are cancelled or rushed. You react to fires instead of developing people. Promotions stall because you lack the time to build cases.

**How to stay in the sweet spot:** When you approach 7–8 reports and the workstreams are naturally diverging, create a sub-team and promote a Tech Lead or Staff Engineer to act as a de facto first-line manager for a cluster (without necessarily giving them the title — this is often called a "TLM" or "Lead" role).

### 4.3 Sub-Team Creation

Sub-teams form when the team's scope exceeds what a single manager and a single mission can cover.

**Signs it is time to split:**
- Stand-ups take longer than 15 minutes.
- The team context-switches between two or more unrelated workstreams.
- A single retro cannot produce a coherent action item because different subgroups have different pain points.
- Knowledge of one part of the system is concentrated in 2–3 people who never work with the rest.
- The manager is a communication bottleneck — decisions that two subgroups could handle locally are escalated.

**How to split well:**

1. **Split by mission, not by technology.** Each sub-team should own an outcome (improve API latency, reduce P95 deploy lead time) rather than a technology (the Kubernetes team, the Go team). Mission-aligned teams develop deeper ownership and make better trade-offs.
2. **Ensure each sub-team has a clear stakeholder or customer.** A sub-team without a user is an ivory tower.
3. **Staff each sub-team with at least one senior engineer.** A sub-team of all juniors struggles with technical decision-making and coaching.
4. **Cluster related sub-teams under a tribe or platform guild** (see [Team Topology](#7-team-topology)) so they retain an identity larger than their sub-team.
5. **Define the interface between sub-teams explicitly.** Who owns shared resources (CI/CD pipeline, test infrastructure, cloud account)? How do they coordinate?

---

## 5. Development

Growing people is the primary responsibility of an engineering manager. If your team members do not improve under your leadership, you are not managing — you are administrating.

### 5.1 Growth Plans

Every team member should have a written growth plan that answers: "What do you want to be true about your career 12–18 months from now, and what are the milestones to get there?"

**Elements of a growth plan:**

- **Current state.** Honest assessment of strengths, growth areas, and blind spots. Written jointly by manager and individual.
- **Target state.** A specific role or capability level. "I want to be a Staff Engineer who owns the observability strategy across all product teams." Vague goals produce vague progress.
- **Milestones (quarterly).** Concrete, observable indicators. "By end of Q3, lead one cross-team incident review and publish the post-mortem. By end of Q4, deliver a talk on our tracing infrastructure at an internal tech forum."
- **Support from the manager.** "I will find you a mentor in the observability space. I will clear your calendar for 4 hours per week to work on the tracing project. I will introduce you to the VP of Platform."

### 5.2 Mentorship

Formal mentorship is one of the most effective development levers. It benefits both the mentor (leadership skills, fresh perspectives) and the mentee (accelerated growth, expanded network).

**Setting up mentorship:**
- **Cross-team is best.** A mentor from a different team brings unbiased perspective and broader organisational context.
- **Structure the relationship.** Monthly 45-minute meetings. A shared document for topics and take-aways. A 6-month trial period with a check-in to assess fit.
- **Train mentors.** Not everyone naturally knows how to mentor. Provide a one-pager: "How to be a mentor" covering active listening, asking vs telling, giving constructive feedback, and spotting growth opportunities.

**Reverse mentorship:** Pair senior leaders with junior team members to build empathy and expose leaders to ground-level realities. For example, a VP paired with a junior engineer learns about CI friction firsthand, and the junior engineer gains exposure to strategic thinking.

### 5.3 Delegation as Development

Delegation is not just about freeing your time — it is the primary mechanism for growing your team. If you never delegate something that makes you uncomfortable, you are capping your team's growth.

**The delegation ladder (from low to high autonomy):**

1. **Tell.** "Do X by Friday."
2. **Sell.** "Here is why X matters. Please do it by Friday."
3. **Consult.** "Here is the problem. What do you think? Here is what I think. You decide."
4. **Delegate.** "Here is the outcome I want. You decide the approach. Keep me informed."
5. **Empower.** "Own this area. Escalate only if you hit a strategic crossroads."

**For each delegation level, be explicit:** "I am delegating this at level 4 — I trust your judgment on approach. Please give me a weekly 2-sentence update unless you get stuck."

**What to delegate:** Not just low-value tasks. Delegate high-visibility, high-stakes work that stretches the individual's capabilities. A junior engineer running a major incident call is terrifying and transformative. Shadow them for the first one, then let them lead.

### 5.4 Promotion Readiness

Promotions should not be a surprise. If someone is surprised they were not promoted, you have failed as a manager.

**System for readiness:**
- **Transparent criteria.** Publish the competencies required for each level (IC3 → IC4 → Staff, etc.). Use these as a shared reference, not a secret checklist.
- **Regular calibration conversations.** Every 1:1 should have a career pulse check. Formally, every quarter, review the growth plan and assess progress against promotion criteria.
- **360-degree evidence gathering.** A promotion case requires data from peers, stakeholders, and cross-functional partners. Do not wait until promo time — ask for feedback throughout the year and keep a brag document or "impact log."
- **Stretch assignments before promotion.** The rule of thumb: someone should be operating at the next level for 3–6 months before they are promoted. This de-risks the promotion and ensures the individual is set up for success.
- **The "no" conversation.** If someone is not ready, be blunt and specific. "You are not ready for Staff because your technical communication does not yet influence cross-team decisions. Here is what we will work on over the next 2 quarters." Undefined expectations are the #1 source of promo dissatisfaction.

---

## 6. Performance Management

Performance management is the process of aligning individual performance with team goals, addressing gaps early, and recognising contributions. It is not an annual cycle — it is a daily practice.

### 6.1 Clear Expectations

**The single biggest cause of underperformance is unclear expectations.** Every team member should be able to answer three questions at any time:

1. What is my top priority this week?
2. What does "done" look like for that priority?
3. How will success be measured?

**Making expectations concrete:**
- Write them down. Verbal expectations are not expectations — they are suggestions. Use OKRs, sprint goals, or a simple "top 3 priorities" doc per quarter.
- Define the "definition of done" for each role. For a platform engineer: "Every PR includes a test, documentation, and a rollback plan. On-call incidents acknowledged within 5 minutes. P1 incidents resolved within 2 hours."
- Align on non-negotiables early. "Everyone participates in on-call rotation. Code reviews are completed within 24 hours during the work week."

### 6.2 Regular 1:1s

The 1:1 is the most important recurring meeting in management. It is the team member's meeting, not the manager's.

**Structure (45–60 minutes, weekly for direct reports):**
- **First 10 minutes:** What is on their mind? Personal updates, frustrations, observations. Let them drive.
- **Next 20 minutes:** Work-in-progress review, blockers, technical decisions needing input. The manager listens and coaches; the goal is not to solve the problem but to help the team member solve it.
- **Last 15 minutes:** Career and growth. "What did you learn this week? What are you excited about? What are you worried about? How can I help you grow?"

**What NOT to do in a 1:1:** Status update reporting (that is what stand-up is for), giving feedback that should have been given immediately, skipping because "nothing is urgent" (the most important conversations happen in non-urgent moments).

### 6.3 Addressing Issues Early

Unaddressed performance issues do not improve with time — they atrophy. The most common mistake new managers make is waiting too long.

**The feedback timeline:**
- **First observation** → same-day private conversation. "I noticed X in the stand-up. Can we talk about it for 2 minutes?"
- **Pattern emerges** (2+ occurrences) → structured feedback in the next 1:1. Use SBI, agree on a change plan.
- **No improvement after 2–3 feedback cycles** → formal Performance Improvement Plan (PIP) with clear, measurable goals, weekly check-ins, and a defined timeline (typically 30–60 days).

**The PIP is not a punishment — it is a rescue attempt.** A well-run PIP either brings someone back to performance or creates an unambiguous paper trail for a separation. Both outcomes are better than letting underperformance fester.

**Signs you waited too long:** Peers start complaining about carrying the load. The team's velocity graph has a visible dip. You dread the 1:1 with that person.

**Documentation discipline:** Write down every performance conversation. "Spoke with Alice about late PR reviews (3 PRs >72h old). Agreed she will review incoming PRs within 24h. Follow-up in 2 weeks." This protects the company, the employee (feedback is traceable), and you (memory fades).

---

## 7. Team Topology

Team topology is the organisational structure through which work flows. The right topology amplifies delivery; the wrong topology creates handoff hell.

### 7.1 The Spotify Model

The Spotify model popularised the **Squad → Tribe → Chapter → Guild** structure. While often romanticised, its core principles are transportable:

- **Squad:** A small (6–10 person), cross-functional, autonomous team owning a mission. A squad has all the skills needed to design, build, test, and ship. Think of it as a mini-startup within the org.
- **Tribe:** A collection of 4–10 squads working in a related domain (e.g. "Payments Tribe"). Tribes create alignment and enable resource sharing without creating a matrix.
- **Chapter:** A horizontal group of engineers with the same specialisation (e.g. "QA Chapter") within a tribe. Chapters are led by a Chapter Lead who handles people development. This provides the career growth structure that squads alone do not.
- **Guild:** A voluntary, cross-tribe community of interest (e.g. "Kubernetes Guild"). Anyone can join. Guilds spread practices organically without redrawing the org chart.

**When the Spotify model works:** The organisation is product-aligned, squads are genuinely autonomous (they own their backlog, architecture, and operations), and the chapter/guild structure is invested in, not just drawn on a slide.

**When it fails:** Squads lack real autonomy (they need approval from a central architect to change a config file). Chapters become bureaucratic reporting lines. Guilds meet once and never again.

### 7.2 Squads and Tribes

**Principles for squad formation:**

- **Mission, not component.** A squad owns a customer-facing outcome ("improve checkout conversion") not a technical component ("the cart service"). This aligns autonomy with value.
- **Size discipline.** 6–10 people. Smaller than 6 and bus risk is dangerous. Larger than 10 and communication overhead grows quadratically. If a squad exceeds 10, split it.
- **Cross-functional by default.** Each squad has engineering, QA, product, and design capability. If your squads share a centralised QA team, you have a queue, not a cross-functional team.
- **End-to-end ownership.** The squad builds, tests, deploys, monitors, and operates its own services. If a squad cannot deploy without another team's approval, it is not a squad — it is a feature team with extra steps.

**Tribe mechanics:**
- Tribes need a Tribe Lead (a senior manager or director) who handles alignment, budget, staffing, and escalations.
- Tribes should have a regular sync (weekly or bi-weekly) where squad leads share progress, surface dependencies, and coordinate cross-squad initiatives.
- Tribes should not exceed ~100 people — beyond that, the Tribe Lead loses touch with individual contributors, and the tribe becomes a department in disguise.

### 7.3 Feature Teams vs Component Teams

| Dimension | Feature Team | Component Team |
|-----------|-------------|----------------|
| Mission | Deliver a user-facing feature | Own a technical component (API, library, data layer) |
| Autonomy | High — owns end-to-end delivery | Low — depends on other teams for integration |
| User empathy | High — directly interacts with product | Low — internal consumer is the "user" |
| Coupling risk | Low — teams are decoupled | High — component team becomes a bottleneck for every feature team that needs a change |
| Reuse | Low (teams may duplicate effort) | High (component is shared) |

**Recommendation:** Start with feature teams. They deliver faster, have clearer ownership, and build deeper user empathy. Introduce component teams only when you see clear reuse value that outweighs the coordination cost — typically at 50+ engineers. Even then, treat component teams as platform teams (see section 8) rather than pure shared-service teams.

---

## 8. Architect Perspective: Platform & Infrastructure Teams

As a Solution Architect building a platform or infrastructure team, you face a distinct set of challenges: your customers are internal engineers, your success is invisible (when things work, no one notices), and you must balance velocity with governance.

### 8.1 Growing a Platform / Infrastructure Team

**Phase 1: The Founder Phase (1–5 people)**
- Everyone is full-stack infra — CI/CD, cloud networking, observability, security.
- No formal specialisation. The team operates as a single unit, responding to whatever is burning.
- Success metric: "Can the product team ship without waiting for us?"

**Phase 2: The T-Shaped Phase (5–15 people)**
- Introduce loose specialisation: one person owns CI/CD, another owns observability, another owns Kubernetes.
- Begin documenting runbooks and standardising patterns (e.g. "every service gets a Helm chart, a Grafana dashboard, and a PagerDuty escalation policy").
- Start a weekly platform office hours or RFC review session where product teams can propose infrastructure changes.
- Success metric: "Are product teams adopting our patterns without hand-holding?"

**Phase 3: The Product Phase (15+ people)**
- Organise into sub-teams aligned to platform domains: Compute & Orchestration, Observability, Developer Experience (DX), Security & Compliance.
- Each sub-team has a product manager (or a tech lead acting as PM) who treats internal developers as customers — surveying them, understanding pain points, prioritising features.
- Introduce SLAs for platform services: "CI pipeline completes in under 10 minutes 95% of the time." "Self-service cluster provisioning within 2 hours."
- Success metric: "Are product teams measurably more productive because of us?"

### 8.2 Balancing Velocity with Governance

The tension between platform teams (who want standardisation, security, and reliability) and product teams (who want to ship fast) is inherent. Managing it is the core skill of a platform architect.

**Principles:**

- **Govern as a product, not a policy.** Instead of "you must use our approved service mesh", provide a self-service interface that makes the right thing the easy thing. If your governance is a form an engineer fills out and waits 3 days for approval, it will be bypassed.
- **Golden paths, not golden handcuffs.** Define and document the "golden path" — the fully supported, well-documented, secure-by-default way to do common tasks (deploy a service, connect to a database, set up monitoring). Teams that stay on the golden path get friction-free support. Teams that want to deviate must justify it — and they bear the operational burden of their deviation.
- **Use gating, not blocking.** Automated policy-as-code checks (e.g. "no public S3 buckets", "container must not run as root") that run in CI are fast and objective. Human approval gates that wait for a ticket to be triaged are slow and subjective. Automate what you can, and only escalate to humans for exceptions.
- **Chargeback or show cost.** If product teams do not feel the cost of their infrastructure decisions, they will over-provision and under-optimise. Implement showback (visible but not charged) or chargeback (real budget impact) to create accountability.
- **Reduce friction relentlessly.** Every time a product engineer says "I could have shipped yesterday if the platform hadn't…", treat that as a P2 incident in your team. Track platform friction as a metric — time from "I need a new service" to "it is running in production with monitoring and a pipeline". Measure it, publish it, improve it.

**Common antipatterns:**
- Building a perfect platform before anyone uses it. Ship a minimal path, get feedback, iterate.
- Saying "no" without offering an alternative. The platform team's job is to enable, not to gatekeep.
- Measuring platform team output (tickets closed, features shipped) instead of outcome (product team velocity, developer satisfaction). An infra team that ships 50 features no one uses is worse than one that ships 5 features everyone loves.
- Treating all product teams the same. A team building the core transaction engine has different infrastructure needs than a team building an internal reporting dashboard. Tier your platform services accordingly — gold, silver, bronze.

---

## Conclusion

Growing a team is not a linear process — it is a cycle of hiring, onboarding, building culture, scaling structure, developing people, managing performance, and evolving topology. No single section of this guide works in isolation. A great hiring pipeline is wasted without strong onboarding. A strong culture will degrade if performance issues are not addressed. A well-designed topology will fail if the team lacks psychological safety.

**Final advice:** Start with the fundamentals — hire deliberately, onboard generously, build trust, set clear expectations, and invest in your people's growth. Do these well, and the structures (team topology, sub-teams, processes) will follow naturally. Do them poorly, and no org chart in the world can save you.

---

*This guide was written by Jack Liu Shurui, Solution Architect at Crédit Agricole CIB, as a living reference for engineering leaders growing their teams. It is maintained in the technology/ directory of the research repository and should be updated as the team evolves and new patterns emerge.*
