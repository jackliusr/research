# Reinforcement Learning Algorithms: A Comprehensive Guide

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore  
> **Context:** Machine Learning / AI — RL Problem Formulation, MDPs, Value-Based Methods, Policy-Based Methods, Actor-Critic, Model-Based RL, RLHF/DPO/GRPO, Multi-Agent RL, Offline RL, Contextual Bandits, Banking Applications  
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)  
> **Last Updated:** August 2026

---

## Table of Contents

1. [What Is Reinforcement Learning?](#1-what-is-reinforcement-learning)
2. [The Formal Framework: Markov Decision Processes](#2-the-formal-framework-markov-decision-processes)
3. [Exploration vs Exploitation: The Fundamental Trade-off](#3-exploration-vs-exploitation-the-fundamental-trade-off)
4. [The Credit Assignment Problem](#4-the-credit-assignment-problem)
5. [Value Functions and the Bellman Equations](#5-value-functions-and-the-bellman-equations)
6. [RL vs Supervised and Unsupervised Learning](#6-rl-vs-supervised-and-unsupervised-learning)
7. [The Algorithm Taxonomy: A Map of the RL Landscape](#7-the-algorithm-taxonomy-a-map-of-the-rl-landscape)
8. [Model-Free Value-Based Methods: Tabular Algorithms](#8-model-free-value-based-methods-tabular-algorithms)
9. [The n-Step and λ-Return Spectrum](#9-the-n-step-and--return-spectrum)
10. [Deep Value-Based Methods: DQN and Its Variants](#10-deep-value-based-methods-dqn-and-its-variants)
11. [Model-Free Policy-Based Methods](#11-model-free-policy-based-methods)
12. [Actor-Critic and Modern Continuous-Control Methods](#12-actor-critic-and-modern-continuous-control-methods)
13. [Model-Based Reinforcement Learning](#13-model-based-reinforcement-learning)
14. [RLHF and Preference-Based RL](#14-rlhf-and-preference-based-rl)
15. [Multi-Agent Reinforcement Learning](#15-multi-agent-reinforcement-learning)
16. [RL in Finance and Banking](#16-rl-in-finance-and-banking)
17. [Offline RL: The Answer for Banking](#17-offline-rl-the-answer-for-banking)
18. [Contextual Bandits: The Single-Step RL](#18-contextual-bandits-the-single-step-rl)
19. [Evaluation and Practical Considerations](#19-evaluation-and-practical-considerations)
20. [When to Use RL: The Decision Framework](#20-when-to-use-rl-the-decision-framework)
21. [The Algorithm Cheat-Sheet](#21-the-algorithm-cheat-sheet)
22. [The Future: 2026 and Beyond](#22-the-future-2026-and-beyond)
23. [Glossary](#23-glossary)
24. [References and Further Reading](#24-references-and-further-reading)

---

## 1. What Is Reinforcement Learning?

### 1.1 Learning by Interaction

**Reinforcement learning (RL)** is the branch of machine learning concerned with how an **agent** should take **actions** in an **environment** to maximize a cumulative numerical **reward** signal. Unlike supervised learning, which learns from a fixed dataset of labeled examples, RL learns *by doing*: the agent interacts with the environment, observes the consequences of its actions, and adjusts its behavior based on the feedback it receives.

The core loop, repeated at every discrete time step `t`, is:

| Step | What happens |
|---|---|
| 1 | The agent observes the environment's **state** `s_t` |
| 2 | The agent selects an **action** `a_t` according to its **policy** `π(a|s)` |
| 3 | The environment returns a **reward** `r_t` and transitions to a new **state** `s_{t+1}` |
| 4 | The agent updates its knowledge (policy, value estimates, or world model) |

```
Agent ── action a_t ──▶ Environment
  ▲                        │
  │                        ▼
  └── state s_{t+1}, reward r_t ──┘
```

The **policy** is the agent's decision rule — a mapping from states to actions. It can be **deterministic** (`a = μ(s)`) or **stochastic** (`a ~ π(·|s)`, a probability distribution over actions). Everything in RL ultimately reduces to finding a good policy.

### 1.2 What Makes RL Different From the Rest of ML

Three properties distinguish RL problems from supervised or unsupervised ones:

- **No labels, only rewards.** There is no teacher telling the agent the correct action. The reward scalar says *how good* the outcome was, not *what* should have been done.
- **Delayed feedback.** The reward for an action may arrive many steps later. A chess move's quality is only revealed at checkmate.
- **The agent's data depends on its own behavior.** The distribution of states the agent sees is a function of the policy it follows — the agent shapes its own training data (non-i.i.d., temporally correlated).

The central object of study is the **return**: the cumulative discounted reward the agent collects over time. The RL problem is to find a policy that maximizes the *expected* return.

### 1.3 A Brief History in Dates

| Year | Milestone |
|---|---|
| 1950s | Trial-and-error learning studied in psychology (Thorndike's law of effect); Bellman formalizes dynamic programming and the Bellman equations |
| 1960s | **MDPs** formalized (Bellman, Howard); first learning automata |
| 1977 | **Werbos** proposes the first policy-gradient-style idea (backpropagation through adaptive critics) |
| 1982–1984 | **Barto, Sutton & Anderson** build the first TD-style learning systems (adaptive critics) |
| 1988 | **TD learning** formalized (Sutton) |
| 1989 | **Q-learning** (Watkins); **Dyna** architecture (Sutton) |
| 1992 | **REINFORCE** policy gradient (Williams) |
| 1994–1998 | **TD-Gammon** (Tesauro) beats top backgammon players; **Sutton & Barto** publish *Reinforcement Learning: An Introduction*; **Sarsa** |
| 2013–2015 | **DQN** plays Atari at superhuman level (Mnih et al., Nature 2015) — the deep RL breakthrough |
| 2016 | **AlphaGo** beats Lee Sedol; **A3C**, **DDPG**, **GAE** published |
| 2017–2018 | **PPO** (OpenAI); **AlphaZero**; **SAC**, **TD3**; Rainbow DQN |
| 2019–2020 | **MuZero** (DeepMind); **CQL** offline RL; DQN-era methods consolidate |
| 2022–2023 | **RLHF** powers InstructGPT/ChatGPT; **DPO** simplifies preference optimization |
| 2024–2025 | **GRPO** and verifiable-reward RL power reasoning models (DeepSeek-R1); RL becomes the core LLM alignment + reasoning tool |

### 1.4 Where RL Sits in This Repo's ML Series

This guide sits alongside the repo's other machine-learning references: `knowledge_distillation_guide.md`, `llm_model_pruning_guide.md`, `deep_learning_frameworks_comparison_guide.md`, and `real_world_ai_projects_guide.md`. Where those cover compression, frameworks, and shipped AI systems, this guide covers the *decision-making* family of ML: algorithms that act. The LLM-alignment half of the story (RLHF/DPO/GRPO) connects directly to `beyond_rag_guide.md` and `hybrid_multi_agent_systems_guide.md`; the banking half connects to `asset_backed_trading_guide.md`, `banking_limits_domain_guide.md`, `financial_fraud_detection_at_scale_guide.md`, and `financial_risk_compliance_systems_guide.md`.

## 2. The Formal Framework: Markov Decision Processes

### 2.1 The MDP Tuple

The formal foundation of RL is the **Markov Decision Process (MDP)**, a mathematical model of sequential decision-making. An MDP is defined by the tuple `(S, A, P, R, γ)`:

| Element | Meaning |
|---|---|
| `S` | **State space** — all possible states the environment can be in |
| `A` | **Action space** — all actions the agent can take (discrete, continuous, or mixed) |
| `P` | **Transition function** — `P(s'|s,a)`, the probability of landing in state `s'` after taking action `a` in state `s` |
| `R` | **Reward function** — `R(s,a)` (or `R(s,a,s')`), the immediate scalar reward for a transition |
| `γ` | **Discount factor** — `0 ≤ γ ≤ 1`, how much the agent values future reward relative to present reward |

The **Markov property** is the key assumption: the future depends on the present state only, not on the history — `P(s_{t+1} | s_t, a_t) = P(s_{t+1} | s_0, a_0, …, s_t, a_t)`. The state must therefore summarize everything relevant about the past. (In practice, agents often work with partial observations — a **POMDP** — but the MDP is the theoretical workhorse.)

### 2.2 Return, Discounting, and Horizon

The agent's objective is expressed through the **return** — the cumulative discounted reward from time `t`:

```
G_t = r_t + γ·r_{t+1} + γ²·r_{t+2} + … = Σ_{k=0}^{∞} γ^k · r_{t+k}
```

Why discount? Three reasons:

- **Economics**: a dollar today is worth more than a dollar tomorrow — discounting matches time-value-of-money thinking (natural for banking).
- **Mathematical convenience**: with `γ < 1` and bounded rewards, the infinite sum converges; undiscounted infinite-horizon returns can diverge.
- **Behavioral**: it encodes *impatience* and limits the influence of far-future (hard-to-predict) outcomes.

The **horizon** is the number of steps the agent plans over: **finite** (episodic tasks like a trading day or a loan lifecycle), **infinite** (continuing tasks like market making), or **discounted-infinite** (the standard compromise). With `γ = 0` the agent is myopic — it only cares about the immediate reward; with `γ → 1` it weighs all future reward equally.

### 2.3 The RL Problem Statement

The agent's behavior is a **policy** `π(a|s)` — the probability of taking action `a` in state `s`. Every policy induces a **value** — the expected return it achieves. The RL problem:

> Find a policy `π*` that maximizes the expected discounted return from every state:
> `π* = argmax_π E[G_t | s_t = s, π]` for all `s`.

This is the same objective as **optimal control** (the term used in engineering and finance), which is why RL and optimal control share deep roots — and why RL maps so naturally onto trading, pricing, and hedging problems.

### 2.4 Episodes vs Continuing Tasks

| Task type | Structure | Examples |
|---|---|---|
| **Episodic** | Interaction splits into episodes with terminal states; return per episode | Chess game, trade execution order, loan workout |
| **Continuing** | Interaction never ends; return is discounted-infinite | Market making, real-time pricing, fraud screening |

### 2.5 When Is an MDP Actually Available?

The MDP is a *specification*, not a gift: in many real problems (trading, banking) the transition probabilities `P` and rewards `R` are **unknown**. The agent must learn them from interaction. This gives the first great split in the algorithm taxonomy (Section 7): **model-based** methods learn/use `P` and `R` explicitly; **model-free** methods skip the model and learn values or policies directly from experience.

## 3. Exploration vs Exploitation: The Fundamental Trade-off

### 3.1 The Dilemma

At every decision point the agent faces a choice: **exploit** — take the action currently believed best, collecting known-good reward — or **explore** — try actions with uncertain outcomes, gathering information that may reveal a better policy. The dilemma: pure exploitation never discovers improvements; pure exploration never collects reward. Every RL algorithm is, at its core, a mechanism for balancing these.

In banking this trade-off is acute and regulated: an exploitative credit policy is safe but may be leaving money on the table; an exploratory one can cause real losses (see Section 16).

### 3.2 The Canonical Exploration Strategies

| Strategy | Mechanism | Notes |
|---|---|---|
| **ε-greedy** | With probability `ε` take a random action; otherwise take `argmax_a Q(s,a)`; decay `ε` over time | Simplest, ubiquitous; ignores *how much* better one action is |
| **UCB (Upper Confidence Bound)** | Pick `a = argmax_a [Q(s,a) + c·√(ln N / n_a)]` — optimistic bonus shrinking with visit count `n_a` | *Optimism in the face of uncertainty*; provably near-optimal regret (lai-robbins-style bounds) |
| **Thompson sampling** | Maintain a posterior over action quality (Bayesian); sample an action's quality from the posterior, pick the best sampled | Naturally balances exploration/exploitation; the practical favorite for bandits |
| **Boltzmann / softmax** | Sample actions with probability `∝ exp(Q(s,a)/τ)`; temperature `τ` controls randomness | Smooth exploration; `τ→0` recovers greedy, `τ→∞` uniform random |
| **Entropy bonus (modern)** | Add `β·H(π(·|s))` to the objective — the policy is *encouraged* to stay stochastic | Used in A2C/PPO/SAC; differentiable, no schedules needed |

### 3.3 Exploration in Deep RL

Deep RL adds two complications. First, with function approximation the agent must explore in a **high-dimensional state space** — random actions rarely produce interesting states (the "hard exploration" problem; solved for games by curiosity bonuses, count-based exploration, or intrinsic rewards). Second, deterministic exploration (adding noise to a deterministic policy, as in DDPG/TD3) is different from stochastic exploration (sampling from a distribution, as in SAC/PPO). **Noisy nets** (learned parameter noise) and **entropy regularization** are the modern tools of choice.

### 3.4 Regret: Measuring Exploration Quality

The exploration/exploitation balance is formally measured by **regret**: the cumulative difference between the reward the optimal policy would have collected and what the agent actually collected. Good exploration algorithms achieve sublinear regret — mistakes early, optimality asymptotically. This is the central performance metric of bandit theory (Section 18) and the reason UCB/Thompson sampling beat ε-greedy in production settings where every mistake costs money.

## 4. The Credit Assignment Problem

### 4.1 Delayed Rewards

Most interesting RL problems have **sparse or delayed rewards**: the reward signal arrives long after the actions that caused it. In chess, the reward (win/lose) arrives at the end of the game — which of the ~40 moves deserves credit? In trade execution, the quality of a passive order's placement is only revealed by the fills and market moves that follow. This is the **credit assignment problem**: *which past action(s) caused this reward, and by how much?*

### 4.2 How Algorithms Answer It

| Algorithm family | Credit assignment mechanism |
|---|---|
| **Monte Carlo** | Wait until the episode ends; credit *all* visited states/actions with the full return — zero bias, but high variance and unusable in continuing tasks |
| **TD learning** | Credit only the *immediate* transition: `r + γV(s')` — bootstrap the rest; low variance, some bias |
| **n-step / λ-return** | Interpolate: credit the last `n` steps (or an exponentially weighted tail via eligibility traces) |
| **Policy gradients (REINFORCE)** | Reinforce entire action sequences with the total return — high variance, which is why modern actor-critics subtract baselines and use advantage estimates |
| **Actor-critic / GAE** | Critic estimates the value baseline; the *advantage* `A(s,a) = Q(s,a) − V(s)` isolates each action's *marginal* contribution — the practical solution in deep RL |

### 4.3 The Shape of the Problem

The credit assignment problem has two halves: **temporal** (which time step caused the reward?) and **structural** (which component of the action — e.g., which leg of a multi-leg trade — caused it?). In multi-agent settings (Section 15) credit assignment extends across *agents*: which agent's action caused the team outcome? Value-based methods handle temporal credit elegantly via bootstrapping; policy-based methods handle structural credit via baselines and critics.

## 5. Value Functions and the Bellman Equations

### 5.1 State-Value and Action-Value

Almost all RL is built on **value functions** — estimates of expected return. There are two:

- **State-value function** `V^π(s) = E[G_t | s_t = s, π]` — the expected return starting from state `s` and following policy `π` thereafter.
- **Action-value function** `Q^π(s,a) = E[G_t | s_t = s, a_t = a, π]` — the expected return starting from state `s`, taking action `a`, then following `π`.

The **optimal value functions** — `V*(s) = max_π V^π(s)` and `Q*(s,a) = max_π Q^π(s,a)` — are the best achievable expected returns. A remarkable fact of MDP theory: if you know `Q*`, the optimal policy is trivial — `π*(s) = argmax_a Q*(s,a)`. **The entire RL problem reduces to estimating `Q*` (or `V*`) well.**

### 5.2 The Bellman Equations

The value functions satisfy recursive consistency equations — the **Bellman equations** (Bellman, 1957). They express the value of the current state as the immediate reward plus the discounted value of the next state:

```
V^π(s)  = Σ_a π(a|s) Σ_{s'} P(s'|s,a) [ R(s,a) + γ·V^π(s') ]
Q^π(s,a) = Σ_{s'} P(s'|s,a) [ R(s,a) + γ·Σ_{a'} π(a'|s') Q^π(s',a') ]

Optimal forms (Bellman optimality equations):
V*(s)  = max_a [ R(s,a) + γ·Σ_{s'} P(s'|s,a)·V*(s') ]
Q*(s,a) = R(s,a) + γ·Σ_{s'} P(s'|s,a)·max_{a'} Q*(s',a')
```

The intuition: **the value of a state is the best immediate reward plus the discounted value of wherever you end up.** The Bellman equations are the backbone of the field — every TD algorithm, DQN, and actor-critic is some form of *iterating on the Bellman equation*.

### 5.3 Why the Bellman Equations Matter

- **Recursive structure** — they turn a global optimization problem into a local, iterable one. This is what makes learning from single transitions possible (TD, Section 8.4).
- **Optimality via fixed point** — under standard conditions, iterating `Q ← TQ` (the Bellman operator is a contraction with modulus `γ`) converges to `Q*`. This is the theoretical guarantee behind Q-learning and DQN.
- **Bootstrapping** — the equations let an estimate update from another estimate, which is both the power (variance reduction, learning without episode ends) and the danger (bias, instability — the deadly triad, Section 10.7) of TD methods.

### 5.4 From Value to Policy

Given `V*` and the model, a greedy policy can be derived; given `Q*`, the policy is greedy with no model needed (`argmax` over actions). This "**generalized policy iteration**" — alternating policy evaluation (estimate values) and policy improvement (act greedily w.r.t. values) — underlies every value-based algorithm from dynamic programming to DQN.

## 6. RL vs Supervised and Unsupervised Learning

### 6.1 The Comparison Table

| Dimension | Supervised | Unsupervised | Reinforcement |
|---|---|---|---|
| **Feedback** | Labels (correct answers) | No labels (structure) | Rewards (scalar, evaluative) |
| **Data** | Fixed i.i.d. dataset | Fixed dataset | Generated *by the agent's own actions*, temporally correlated |
| **Goal** | Map inputs → outputs correctly | Find structure/patterns | Maximize cumulative reward over time |
| **Error signal** | Loss vs ground truth | Reconstruction/divergence | Temporal difference / policy gradient |
| **Feedback timing** | Immediate (per example) | Immediate | Often delayed (credit assignment) |
| **Distribution** | Stationary | Stationary | Non-stationary — the agent changes its own data distribution |
| **Learning mode** | Batch | Batch | Online / incremental (or batch via offline RL) |

### 6.2 What This Means in Practice

The differences drive real engineering consequences:

- **No labels → reward design is everything.** In supervised learning the loss is given; in RL the reward function is a design choice, and a bad one yields a policy that optimizes the wrong thing (reward hacking, Section 19.4).
- **Non-i.i.d. data → experience replay.** Deep RL agents re-sampling from a replay buffer (Section 10.3) are partially *recovering* the i.i.d. assumption that supervised learning gets for free.
- **Delayed feedback → bootstrapping.** The value-function machinery (Section 5) exists because you cannot wait for labels that never arrive.
- **Online learning → drift and stability.** The agent must keep learning from a shifting distribution — the reason RL is famously harder to train and evaluate than supervised models (Section 19).

### 6.3 The Hybrid View

Modern practice blurs the line: **offline RL** (Section 17) is supervised-style *batch* learning on logged data; **RLHF** (Section 14) is supervised learning of a reward model followed by RL against it; **Decision Transformer** (Section 17.5) literally reformulates RL as a supervised sequence-prediction problem. The useful mental model: RL is what you reach for when *sequential decisions + delayed rewards* are the essence of the problem — the supervised toolkit handles everything else.
## 7. The Algorithm Taxonomy: A Map of the RL Landscape

### 7.1 The Three Axes

Every RL algorithm can be located along three independent axes. Understanding these axes is the single most useful skill for navigating the field — and for picking an algorithm for a banking problem.

**Axis 1 — Model-free vs model-based.** Does the agent learn/use an explicit model of the environment dynamics `P(s'|s,a)` (and reward)?

- **Model-free** algorithms learn values or policies directly from experience, never modeling transitions. They dominate practice (DQN, PPO, SAC) because no model is needed — but they are sample-hungry.
- **Model-based** algorithms learn a dynamics model, then plan or optimize inside it (Dyna, MuZero, Dreamer). Sample-efficient, but the model's errors bias the solution.

**Axis 2 — Value-based vs policy-based vs actor-critic.** *What* is learned?

- **Value-based**: learn `Q(s,a)` or `V(s)`; derive the policy implicitly (greedy w.r.t. values). Discrete action spaces only (`argmax`).
- **Policy-based**: learn the policy `π_θ(a|s)` directly via gradient ascent on expected return. Natural for continuous actions; can learn stochastic policies.
- **Actor-critic**: learn *both* — the actor (policy) and the critic (value function used as a baseline/advantage estimator). The dominant modern paradigm.

**Axis 3 — On-policy vs off-policy.** *Whose data* is the agent learning from?

- **On-policy**: learn only from data generated by the *current* policy. Safe (no stale-data surprises) but sample-inefficient — every update invalidates the old data (PPO, A2C, REINFORCE).
- **Off-policy**: learn from *any* data — including old data in a **replay buffer** and data from other policies. Sample-efficient, data-reusing, but trickier to stabilize (DQN, SAC, TD3, Q-learning).

### 7.2 The Landscape Map

```
                        REINFORCEMENT LEARNING
                                │
              ┌─────────────────┴─────────────────┐
        MODEL-FREE                           MODEL-BASED
    (learn from experience)               (learn a dynamics model)
              │                                  │
   ┌──────────┼──────────┐              Dyna-Q, MBPO, MuZero,
 VALUE-    POLICY-    ACTOR-            Dreamer, World Models
 BASED     BASED      CRITIC
   │          │          │
 Tabular:  REINFORCE   A2C/A3C
 DP, MC,   vanilla PG  PPO (on-policy)
 TD, SARSA     │        SAC, TD3, DDPG
 Q-learning    └──────► (off-policy)
   │
 Deep: DQN → Double → Dueling → PER → Rainbow → C51/QR-DQN
   │
 Offline RL: BCQ, CQL, IQL, Decision Transformer
   │
 Bandits (horizon-1 RL): ε-greedy, UCB, Thompson, LinUCB
```

### 7.3 The Taxonomy Table

| Algorithm | Axis 1 | Axis 2 | Axis 3 | Action space |
|---|---|---|---|---|
| Policy / value iteration | Model-based (given model) | Value-based | — (planning) | Discrete |
| Monte Carlo | Model-free | Value-based | Off-policy possible | Discrete |
| SARSA | Model-free | Value-based | On-policy | Discrete |
| Q-learning | Model-free | Value-based | Off-policy | Discrete |
| DQN & variants | Model-free | Value-based | Off-policy | Discrete |
| REINFORCE | Model-free | Policy-based | On-policy | Both |
| A2C / A3C | Model-free | Actor-critic | On-policy | Both |
| PPO / TRPO | Model-free | Actor-critic | On-policy | Both |
| DDPG / TD3 | Model-free | Actor-critic | Off-policy | Continuous |
| SAC | Model-free | Actor-critic | Off-policy | Continuous |
| Dyna-Q | Model-based | Value-based | Off-policy | Discrete |
| MuZero | Model-based | Actor-critic + MCTS | Off-policy | Discrete (and continuous) |
| CQL / IQL / BCQ | Model-free (offline) | Value-based / actor-critic | Offline (batch) | Both |
| Decision Transformer | Model-free (offline) | Sequence model | Offline | Both |
| PPO/GRPO (RLHF) | Model-free | Policy-based (no critic in GRPO) | On-policy | Discrete tokens |

### 7.4 How to Read the Map

- **Sample efficiency** increases left-to-right across the model axis: model-based > off-policy model-free > on-policy model-free (details in Section 13.5).
- **Stability** is inversely correlated with off-policy-ness: on-policy methods are easier to stabilize; off-policy deep methods need tricks (target networks, double Q, clipping).
- **Action space** is the hard constraint: if actions are continuous (portfolio weights, order sizes, prices), value-based `argmax` methods are out — reach for policy-based or actor-critic.
- **Data regime** decides everything in banking: with only historical logs (no live interaction), you are restricted to the offline column (Section 17).

## 8. Model-Free Value-Based Methods: Tabular Algorithms

### 8.1 Dynamic Programming: The Theoretical Foundation

**Dynamic programming (DP)** solves MDPs *given a known model* (`P` and `R` known). It is not a learning method — it is the theoretical foundation every learning method approximates:

- **Policy iteration**: alternate (1) *policy evaluation* — iteratively apply the Bellman equation for a fixed policy until `V^π` converges; (2) *policy improvement* — make the policy greedy w.r.t. `V^π`. Repeat until no improvement.
- **Value iteration**: merge the two — repeatedly apply the Bellman *optimality* equation directly: `V(s) ← max_a Σ_{s'} P(s'|s,a)[R + γV(s')]`, until convergence.

DP establishes the key results the rest of the field leans on: Bellman-operator contraction (guaranteed convergence with rate `γ`), and **generalized policy iteration** as the universal skeleton. Its limitation is the assumption that `P` and `R` are known — in real problems they are not, which motivates sampling-based learning.

### 8.2 Monte Carlo Methods: Learning From Episodes

**Monte Carlo (MC)** methods learn value functions from *complete episodes*, averaging the actual returns observed. They need no model and no bootstrapping:

- **First-visit MC**: for each state, average the return following its *first* occurrence in an episode.
- **Every-visit MC**: average the return following *every* occurrence.

MC is **unbiased** (it averages real returns) but **high variance** (returns depend on all the randomness in the episode), and it only works for episodic tasks. It is the "wait until the end" end of the bias-variance spectrum (Section 9).

### 8.3 Temporal Difference Learning: TD(0)

**TD learning** (Sutton, 1988) is the key idea that makes RL practical: learn from *single transitions* by **bootstrapping** — using the current estimate of the next state's value in place of the true future:

```
V(s) ← V(s) + α [ r + γ·V(s') − V(s) ]
```

The bracketed term is the **TD error** `δ = r + γV(s') − V(s)` — the difference between the *predicted* value and the *observed* value-plus-next-prediction. Properties:

- **Learns online**, after every step — no need to wait for episode end (works in continuing tasks).
- **Lower variance than MC** (only one transition's randomness enters) at the cost of **bias** (the bootstrap target uses estimates).
- The simplest instantiation of the "predict, observe, correct" loop that defines the whole field.

### 8.4 SARSA: On-Policy TD Control

Extending TD from state values to action values gives **control** (finding the optimal policy). **SARSA** (State-Action-Reward-State-Action) is the on-policy version — it updates using the action *actually taken* in the next state:

```
Q(s,a) ← Q(s,a) + α [ r + γ·Q(s',a') − Q(s,a) ]
```

Because `a'` comes from the current policy (typically ε-greedy), SARSA learns the value of the *exploration policy* itself. Consequence: SARSA learns **conservative** policies — it internalizes the risk of exploratory actions. In an environment where a bad slip is costly (trading with a risk limit), on-policy conservatism can be a feature, not a bug.

### 8.5 Q-Learning: Off-Policy TD Control

**Q-learning** (Watkins, 1989) is the most influential tabular algorithm in the field — the foundation of DQN and the entire deep value-based lineage. It updates with the *greedy* (max) next-action value regardless of which action the behavior policy actually takes:

```
Q(s,a) ← Q(s,a) + α [ r + γ·max_{a'} Q(s',a') − Q(s,a) ]
```

Q-learning is **off-policy**: it can learn the optimal policy from data generated by *any* policy — including a highly exploratory one, or (in modern form) a replay buffer of old data. This decoupling of behavior from target is exactly what makes experience replay and offline RL possible. Under standard conditions Q-learning converges to `Q*`; its empirical flaw is **overestimation bias** (the `max` over noisy estimates inflates values), the bug that Double DQN later fixed (Section 10.4).

### 8.6 Tabular Methods in the Real World

| Method | Model needed? | Bootstraps? | On/off-policy | Use when |
|---|---|---|---|---|
| Policy/value iteration | Yes | Yes | — | Small MDPs, planning, theory |
| Monte Carlo | No | No | Off-policy possible | Episodic, small state spaces |
| TD(0) | No | Yes | Both | Prediction, online learning |
| SARSA | No | Yes | On-policy | Risk-averse control, live exploration |
| Q-learning | No | Yes | Off-policy | Discrete control, replay-friendly |

Tabular methods require enumerable states — they break when the state space is large (prices, portfolios, customer histories). That is the gap deep function approximation fills. But the *principles* — TD error, bootstrapping, off-policy updates, generalized policy iteration — carry over unchanged into deep RL.

## 9. The n-Step and λ-Return Spectrum

### 9.1 TD(0) ↔ Monte Carlo Continuum

TD(0) and Monte Carlo are the two extremes of a spectrum:

| Method | Target for `V(s_t)` | Bias | Variance | When it can learn |
|---|---|---|---|---|
| MC (∞-step) | Full episode return `G_t` | None | High | Episode end |
| TD(0) (1-step) | `r_t + γV(s_{t+1})` | Some | Low | Every step |
| n-step TD | `r_t + γr_{t+1} + … + γ^{n-1}r_{t+n-1} + γ^n V(s_{t+n})` | Intermediate | Intermediate | After n steps |

**n-step returns** interpolate: look ahead `n` real rewards, then bootstrap. Larger `n` → less bias, more variance; smaller `n` → the reverse. This is the classic bias-variance trade-off made concrete, and it shows up everywhere in modern practice — multi-step returns are one of the six Rainbow components (Section 10.6) and are standard in A2C/PPO.

### 9.2 TD(λ) and Eligibility Traces

**TD(λ)** generalizes the spectrum to a *weighted average of all n-step returns* — the **λ-return** — where λ ∈ [0,1] exponentially weights longer horizons. **Eligibility traces** implement this efficiently online: each visited state/action accumulates a decaying trace `e ← γλe + 1`, and *every* recently visited state gets updated proportionally to its trace at each step:

```
δ = r + γV(s') − V(s)
V(s) ← V(s) + α·δ·e(s)  for all s with e(s) ≠ 0
```

- `λ = 0` recovers TD(0); `λ = 1` recovers Monte Carlo.
- Traces solve the *temporal credit assignment* of Section 4 — a state's update "remembers" how recently it was visited.
- In deep RL, exact traces are rarely used (function approximation complicates them); their role is largely taken by **GAE** (Section 12.6), which is TD(λ) for *advantage estimation*.

## 10. Deep Value-Based Methods: DQN and Its Variants

### 10.1 DQN: The Deep RL Breakthrough

**DQN — Deep Q-Network** (Mnih et al., Nature 2015, *"Human-level control through deep reinforcement learning"*) combined Q-learning with a deep CNN and achieved superhuman performance on 49 Atari games from raw pixels — the moment deep RL became a field. The architecture:

- State `s` (game frames) → **CNN** → `Q(s,a)` for each discrete action.
- Loss: `L(θ) = E[(r + γ·max_{a'} Q(s',a'; θ⁻) − Q(s,a; θ))²]` — a regression toward the Q-learning target.
- The policy is `argmax_a Q(s,a)` with ε-greedy exploration.

Naively combining Q-learning with neural networks was known to diverge (the deadly triad, Section 10.7). DQN's two innovations made it work:

### 10.2 Innovation 1: Experience Replay

Store every transition `(s, a, r, s')` in a **replay buffer**; train on *minibatches sampled uniformly* from it. This solves two problems at once:

- **Breaks temporal correlation** — consecutive transitions are highly correlated; random minibatches restore the i.i.d. assumption gradient descent needs.
- **Reuses data** — each transition is used many times, improving sample efficiency and enabling off-policy learning.

### 10.3 Innovation 2: Target Network

The Q-learning target uses the *same network being trained* (`max_{a'} Q(s',a')`), which chases a moving goalpost and can oscillate. DQN keeps a frozen copy — the **target network** `θ⁻` — updated slowly (periodic hard copy, or soft Polyak averaging `θ⁻ ← τθ + (1−τ)θ⁻`). The target's parameters lag, stabilizing the regression: the agent chases a slowly-moving target instead of its own tail.

### 10.4 DQN Variants: Fixing the Flaws

| Variant | Year | Problem fixed | Mechanism |
|---|---|---|---|
| **Double DQN** | van Hasselt et al. 2016 | Overestimation bias (`max` over noisy Q inflates values) | Decouple selection from evaluation: `r + γ·Q(s', argmax_a Q(s',a; θ); θ⁻)` |
| **Dueling DQN** | Wang et al. 2016 | Q conflates "how good is this state" with "how good is this action" | Split streams: `Q(s,a) = V(s) + A(s,a) − mean_a A(s,a)`; better value learning in states where actions don't matter |
| **Prioritized Experience Replay (PER)** | Schaul et al. 2016 | Uniform replay wastes samples on boring transitions | Sample with probability ∝ |TD error|^α; learn more from surprising transitions |
| **Multi-step (n-step) returns** | — | Slow credit assignment of 1-step targets | Use n-step targets (Section 9) |
| **Distributional (C51)** | Bellemare et al. 2017 | Expectation discards risk information | Learn the full *distribution* of returns per (s,a) (51 atoms), minimize distributional divergence |
| **Noisy Nets** | Fortunato et al. 2017 | ε-greedy explores randomly, not systematically | Learned parameter noise; exploration as a learned behavior |

### 10.5 Rainbow

**Rainbow** (Hessel et al., 2017) combined all six improvements — Double, Dueling, PER, multi-step, distributional (C51), Noisy Nets — into one agent. The result substantially beat every individual component on Atari. The lesson: these are largely *orthogonal* fixes, and in practice **multi-step + Double + PER** is a high-value subset; distributional and noisy nets add the rest.

### 10.6 Distributional RL: Beyond the Expectation

Distributional methods (C51, **QR-DQN** quantile regression, **IQN** implicit quantile networks) learn the full return distribution `Z(s,a)` instead of its mean `Q(s,a)`. Why it matters for finance:

- **Risk sensitivity** — the distribution captures tail risk (VaR-style thinking): two actions can have equal expected value but very different loss distributions.
- **Better learning signal** — distributional targets carry more information per transition; C51's improvement was partly a representation win, not just a risk win.

### 10.7 Value-Based Practical Notes

- **Discrete action spaces only** — the policy is `argmax_a Q(s,a)`; continuous actions need a different family (Sections 11–12).
- **The deadly triad** — function approximation + bootstrapping + off-policy is provably unstable in general; DQN's tricks (replay, target nets, Double Q) are exactly the mitigation stack. When DQN diverges, suspect the triad first.
- **Overestimation is structural** — the `max` operator propagates positive noise; always use Double-style decoupling in production.
- **Reward scaling matters** — clip/normalize rewards before they flow into targets; DQN is sensitive to reward magnitude.

## 11. Model-Free Policy-Based Methods

### 11.1 The Policy Gradient Theorem

Value-based methods learn values and derive the policy; **policy-based methods** optimize the policy `π_θ(a|s)` *directly* by gradient ascent on expected return `J(θ)`. The foundation is the **policy gradient theorem** (Sutton et al., 1999):

```
∇_θ J(θ) = E_{s~d^π, a~π_θ} [ ∇_θ log π_θ(a|s) · A^π(s,a) ]
```

The intuition: `∇_θ log π_θ(a|s)` is the direction that increases the probability of action `a` in state `s`; weighting by the **advantage** `A^π(s,a) = Q^π(s,a) − V^π(s)` makes actions *better than average* more probable and actions *worse than average* less probable. The theorem is what makes this weighted likelihood ascent a valid gradient of the return — no model, no value-function argmax required.

### 11.2 REINFORCE: The Monte Carlo Policy Gradient

**REINFORCE** (Williams, 1992) is the simplest policy gradient: use the full return `G_t` as the advantage estimate, over complete episodes:

```
∇_θ J(θ) ≈ Σ_t ∇_θ log π_θ(a_t|s_t) · G_t
```

Simple and unbiased, but **high variance** — whole-episode returns are noisy, and gradients can swing wildly. REINFORCE with a baseline subtracts a state-value estimate: `∇ log π(a|s) · (G_t − V(s))` — variance reduction without bias, because the baseline is constant w.r.t. the action. This baseline idea is the seed of all actor-critics.

### 11.3 Variance Reduction: Baselines and Advantage Functions

The advantage `A(s,a) = Q(s,a) − V(s)` is the *right* weighting: it measures an action's quality *relative to the state's average*. Using `Q` alone leaves the gradient's variance polluted by state-to-state value differences that policy updates shouldn't pay for. Modern practice:

- Critic network estimates `V(s)` (or `Q`) — the **baseline**.
- Advantage estimated by `A = Q − V`, or more cheaply by the TD residual `r + γV(s') − V(s)` (one-sample estimate), or by **GAE** (Section 12.6) for the full λ-weighted version.
- **Entropy regularization** — add `β·H(π(·|s))` to the objective to prevent premature determinism (exploration, Section 3.2).

### 11.4 Vanilla PG → A2C/A3C

**A3C — Asynchronous Advantage Actor-Critic** (Mnih et al., 2016) scaled policy gradients by running many **parallel workers** (each with its own environment copy) asynchronously, sharing a global network. The parallelism decorrelates the data (a cheap stand-in for replay) and speeds wall-clock training dramatically. **A2C** is the synchronous simplification — workers gather rollouts in parallel, then update together — which is more GPU-friendly and reproducible, and is what most libraries ship.

The canonical A2C update per step:

```
δ = r + γV(s') − V(s)                     (TD residual)
θ_actor  ← θ_actor  + α·∇log π(a|s)·δ + β·∇H(π(·|s))   (n-step returns + entropy bonus)
θ_critic ← θ_critic + α·δ·∇V(s)
```

### 11.5 TRPO: The Trust-Region Predecessor

**TRPO — Trust Region Policy Optimization** (Schulman et al., 2015) attacked the policy gradient's core instability: a big update in parameter space can be a catastrophic change in policy space. TRPO enforces a hard constraint — the new policy's **KL divergence** from the old policy must stay under a budget `δ` — guaranteeing monotonic improvement *in theory*:

```
maximize E[ (π_θ/π_old)·A ]   subject to   E[ KL(π_old ‖ π_θ) ] ≤ δ
```

The KL constraint is a **trust region**: the update is only trusted within a neighborhood where the advantage estimates remain valid. TRPO works but is complicated (conjugate gradient, line search); PPO is its practical successor.

### 11.6 PPO: The Default Modern On-Policy Algorithm

**PPO — Proximal Policy Optimization** (Schulman et al., 2017) achieves TRPO-like stability with a first-order objective. Define the **importance ratio** `r_t(θ) = π_θ(a_t|s_t) / π_old(a_t|s_t)` — how much more/less likely the new policy is to take the observed action. The **clipped surrogate objective**:

```
L(θ) = E_t [ min( r_t(θ)·A_t,  clip(r_t(θ), 1−ε, 1+ε)·A_t ) ]
```

The mechanics: when `A_t > 0` (good action), the objective pushes `r_t` up — but the clip at `1+ε` caps how far, so a single minibatch can't destroy the policy. When `A_t < 0`, the clip at `1−ε` similarly bounds the damage. The result is a *soft trust region* — updates that are stable without a hard KL constraint. PPO also typically uses **multiple epochs of minibatch SGD per collected rollout** (reusing on-policy data a few times), which is part of its practical efficiency.

**Why PPO won**: simple to implement, robust across tasks, works with discrete and continuous actions, and its hyperparameters are forgiving. It is the default on-policy algorithm for games, robotics, and — decisively — **RLHF**: OpenAI's InstructGPT/ChatGPT pipeline (and most of the industry's) used PPO against a learned reward model (Section 14). It is also the standard baseline for trading and portfolio RL research.

### 11.7 Policy-Based Practical Notes

- **Continuous action spaces are natural** — Gaussian policies (mean + std from the network, sample actions) handle any dimensionality; no argmax needed.
- **On-policy = sample-inefficient** — new data must be collected for every update; PPO needs millions of environment steps for hard tasks. In banking terms: you cannot afford this online (Section 16.7), which is why offline RL exists (Section 17).
- **High variance** — mitigate with baselines/advantage estimates (GAE), entropy regularization, gradient clipping, and trust-region/clipping mechanisms.
- **Local optima** — policy gradients can converge prematurely to poor local optima; reward shaping and exploration bonuses help.
## 12. Actor-Critic and Modern Continuous-Control Methods

### 12.1 The Actor-Critic Family

The **actor-critic** architecture is the dominant paradigm in modern deep RL: an **actor** (the policy `π_θ` that chooses actions) and a **critic** (a value function `V_φ` or `Q_φ` that judges them). The critic's value estimate serves as the baseline/advantage estimator that tames policy-gradient variance (Section 11.3); the actor improves against the critic's judgment:

```
Critic:  learns Q(s,a) or V(s)  →  provides advantage estimates A(s,a)
Actor:   learns π_θ(a|s)        →  updated by ∇log π(a|s)·A(s,a)
```

A2C and PPO are actor-critic (the critic is the baseline); DDPG, TD3, and SAC are off-policy actor-critics for continuous control. Everything in this section is actor-critic.

### 12.2 DDPG: Deep Deterministic Policy Gradient

**DDPG** (Lillicrap et al., 2016) extended DQN's off-policy machinery to **continuous action spaces** by learning a *deterministic* policy `a = μ_θ(s)`:

- **Critic** `Q_φ(s,a)` trained by Q-learning-style regression with a target network: `y = r + γ·Q_φ⁻(s', μ_θ⁻(s'))`.
- **Actor** `μ_θ` trained by gradient ascent on `∇_θ J ≈ E[∇_a Q_φ(s,a)|_{a=μ(s)} · ∇_θ μ_θ(s)]` — push actions toward whatever raises the critic's estimate.
- **Replay buffer + target networks** for both actor and critic (soft Polyak updates); exploration via **noise added to the deterministic action** (Ornstein-Uhlenbeck or plain Gaussian).

DDPG works but is notoriously brittle: the deterministic policy's value estimates overestimate (same `max`-style bias), and actor-critic coupling can oscillate.

### 12.3 TD3: Twin Delayed DDPG

**TD3** (Fujimoto et al., 2018) is DDPG made robust, fixing three known failure modes:

| Fix | Mechanism |
|---|---|
| **Clipped double-Q** | Two critics; target uses `min(Q_1, Q_2)` — the pessimistic of the two kills overestimation bias |
| **Delayed policy updates** | Update the actor (and target networks) less often than the critic — the critic gets time to become accurate first |
| **Target policy smoothing** | Add noise to the target action: `a' = μ(s') + clip(ε, −c, c)` — the critic learns that similar actions have similar values, resisting sharp adversarial peaks |

TD3 is the reliable, off-the-shelf deterministic continuous-control algorithm — a common default for portfolio-rebalancing research.

### 12.4 SAC: Soft Actor-Critic — The State of the Art

**SAC** (Haarnoja et al., 2018) is the default off-policy algorithm for continuous control. Its defining idea is **maximum entropy RL**: maximize the expected return *plus* the policy's entropy:

```
J(π) = Σ_t E[ r(s_t,a_t) + α·H(π(·|s_t)) ]
```

- **Stochastic policy**: actions are *sampled* from a distribution (usually squashed Gaussian), so exploration is built into the policy rather than bolted on as noise.
- **Soft Q-learning**: the critic is trained with a soft target that includes the entropy term — the agent explicitly values having multiple good options.
- **Auto-tuned temperature `α`**: a learned Lagrange multiplier adjusts how much exploration pressure the policy carries, so the entropy bonus doesn't need manual scheduling.
- **Double-Q critics + replay buffer + target networks**: inherits TD3's stability machinery.

SAC typically matches or beats TD3/DDPG on continuous benchmarks while exploring more sensibly — which is why it is the go-to for anything with continuous actions, including finance research (execution, allocation) where a *distribution* of actions is also useful for representational exploration.

### 12.5 The Off-Policy Continuous Control Comparison

| Algorithm | Policy | Q-learning style | Exploration | Stability | Sample efficiency |
|---|---|---|---|---|---|
| DDPG (2016) | Deterministic | Single Q | Action noise | Fragile | Medium |
| TD3 (2018) | Deterministic | Clipped double Q | Action noise + smoothing | Robust | Medium |
| SAC (2018) | Stochastic (max-entropy) | Double Q + entropy | In-policy sampling | Robust | High |

### 12.6 GAE: Generalized Advantage Estimation

**GAE** (Schulman et al., 2016) is the standard way to compute the advantages that feed actor-critic updates. It is TD(λ) applied to advantages: for each step compute the TD residual `δ_t = r_t + γV(s_{t+1}) − V(s_t)`, then exponentially weight n-step advantage estimates:

```
Â_t = Σ_{l=0}^{∞} (γλ)^l · δ_{t+l}
    = δ_t + γλ·δ_{t+1} + (γλ)²·δ_{t+2} + …
```

- `λ = 0` → 1-step (TD-style) advantages: low variance, high bias.
- `λ = 1` → full Monte-Carlo advantages: low bias, high variance.
- A middle `λ ≈ 0.95` is the practical default in PPO/A2C/SAC-style pipelines.

GAE is the single most-used advantage estimator in modern deep RL — a small knob (`λ`) that trades bias and variance exactly where the user needs it.

### 12.7 Choosing Among the Modern Methods

| Situation | Algorithm |
|---|---|
| Discrete actions, off-policy, replay-friendly | DQN family / Rainbow |
| Continuous actions, off-policy, sample budget matters | **SAC** (default), TD3 (deterministic) |
| Continuous or discrete, on-policy, parallel envs available | **PPO** (default), A2C |
| Small discrete tabular problems | Q-learning / SARSA |
| Very high sample efficiency demanded | Model-based (Section 13) |

## 13. Model-Based Reinforcement Learning

### 13.1 The Model-Based Idea

**Model-based RL** learns an explicit model of the environment — transition dynamics `P̂(s'|s,a)` and reward `R̂(s,a)` — and then *plans or optimizes inside it*. The learned model lets the agent **imagine** experience: train a policy with simulated rollouts, use the model for planning at decision time, or both. The payoff is **sample efficiency** — real interactions are the expensive resource, and a good model multiplies them.

### 13.2 The Model-Based Lineage

| Method | Year | Idea |
|---|---|---|
| **Dyna-Q** | Sutton 1990 | Interleave real Q-learning updates with *imagined* updates from the learned model — the classic model + value-learning hybrid |
| **MBPO** | Janner et al. 2019 | Train a policy with short-horizon rollouts from an ensemble of learned models (short horizons avoid compounding error) |
| **World Models** | Ha & Schmidhuber 2018 | Learn a compact latent representation + dynamics (a small "world"), train a controller inside it — CarRacing from a tiny model |
| **Dreamer** | Hafner et al. 2020–2023 | Latent world model (RSSM) + actor-critic trained entirely in imagination; visual control from pixels; DreamerV2/V3 improved the latent model and are state-of-the-art in model-based visual control |
| **AlphaGo → AlphaZero** | 2016/2018 | MCTS + self-play with *known* game dynamics; the planner uses a learned value/policy network to guide tree search |
| **MuZero** | Schrittwieser et al. 2020 | Removes the need for known dynamics: learns the transition model *itself* (in latent space) and plans with MCTS — AlphaZero's planning + learned model; beat Atari and chess/shogi/Go without a simulator |

### 13.3 How MuZero Works

MuZero's signature: the agent learns a **latent dynamics model** — `h(s_t) → z_t`, `g(z, a) → z'` — plus a value predictor and a policy prior, and plans with **Monte Carlo Tree Search (MCTS)** in latent space at every decision. The model only needs to be accurate *where planning looks*, not everywhere — which is why it stays sample-efficient without a simulator. For games it is superhuman; for control problems it demonstrates the sample-efficiency frontier of model-based RL.

### 13.4 Model-Based Pros and Cons

| Pros | Cons |
|---|---|
| **Sample efficiency** — learns from orders of magnitude fewer real interactions (huge in real-world settings like trading, where real episodes are scarce/expensive) | **Model bias** — the model is wrong somewhere; errors compound over rollout horizons (the "compounding error" problem) |
| Can plan *before* acting (safely, in imagination) | Model learning is a second hard learning problem (distribution shift: the policy explores regions the model never saw) |
| Naturally handles risk: simulate many futures, pick the robust plan | More complex engineering (model + planner + policy) |
| The model itself is a reusable asset (scenario generation, what-if analysis) | Overfitting the model to training data can silently invalidate the policy |

### 13.5 The Sample-Efficiency Spectrum

```
sample efficiency (real interactions needed per unit of learning)
   low ◀──────────────────────────────────────────────▶ high

  on-policy        off-policy           model-based
  model-free       model-free           (MuZero, Dreamer)
  (PPO, A2C)       (SAC, TD3, DQN)      + offline RL on logs
   millions         hundreds of              tens of
   of steps         thousands                thousands
```

Rough practical magnitudes (benchmark tasks): on-policy PPO needs millions of environment steps; off-policy SAC/TD3 needs hundreds of thousands; model-based MuZero/Dreamer and offline RL on good logged data can succeed with tens of thousands of *real* interactions. In banking terms: on-policy online RL is effectively impossible (real market steps are the environment, and you cannot run millions of them live); off-policy and model-based approaches make simulation-based projects feasible; offline RL (Section 17) removes live interaction entirely.

## 14. RLHF and Preference-Based RL

### 14.1 The LLM Alignment Problem

The modern LLM era gave RL its biggest deployment yet. A pretrained language model predicts the next token well but does not necessarily produce *helpful, honest, harmless* responses. **RLHF — Reinforcement Learning from Human Feedback** (InstructGPT, Ouyang et al., 2022; the method behind ChatGPT) aligns the model to human preferences. The pipeline has three stages:

```
Stage 1: SFT        — supervised fine-tuning on high-quality demonstrations
Stage 2: Reward model — train R_φ(x,y) on human preference comparisons (Bradley-Terry)
Stage 3: RL         — optimize the policy against R_φ with PPO (KL-regularized)
```

### 14.2 Stage 2: Training the Reward Model

Humans can't reliably give scalar scores, but they *can* rank pairs. Collect a dataset of comparisons — `(x, y_w, y_l)` where `y_w` is preferred over `y_l` — and fit the reward model with the **Bradley-Terry** model of pairwise preferences:

```
P(y_w ≻ y_l | x) = σ( R_φ(x, y_w) − R_φ(x, y_l) )   (σ = sigmoid)
loss: − log σ( R_φ(x, y_w) − R_φ(x, y_l) )
```

The reward model is itself a transformer (often the SFT model with the head removed) and is the *learned reward function* the RL stage optimizes.

### 14.3 Stage 3: PPO Against the Reward Model

The policy (the chat model) is optimized with **PPO** to maximize reward-model score, with two crucial stabilizers:

- **KL penalty**: `r_total = R_φ(x,y) − β·KL(π_θ ‖ π_ref)` — keeps the policy near the SFT reference so it doesn't drift into reward-model exploitation (gibberish that scores high).
- The policy generates responses, the reward model scores them, and PPO updates the policy — the standard on-policy loop, at token-sequence scale.

RLHF is why PPO is arguably the most-deployed RL algorithm in history: GPT-3.5/ChatGPT and Claude's early alignment used PPO-based RLHF (Claude moved to RLAIF/Constitutional AI variants over time).

### 14.4 DPO: Direct Preference Optimization

**DPO** (Rafailov et al., 2023) showed the reward model is removable: the RLHF objective has a closed-form solution relating the optimal policy to the reward, so the policy can be optimized *directly* on preference data:

```
L_DPO(θ) = −E[ log σ( β·log(π_θ(y_w|x)/π_ref(y_w|x)) − β·log(π_θ(y_l|x)/π_ref(y_l|x)) ) ]
```

Intuition: the loss simply raises the *relative log-probability* of preferred completions over dispreferred ones, implicitly fitting a reward model *inside* the policy update. DPO needs no reward model, no PPO loop, no sampling of rollouts — it is a classification-style loss on static preference data. That makes it simpler, cheaper, and dramatically more stable, at the cost of less control over the optimization (no explicit KL schedule, no online data collection). DPO and its variants are now the default for preference alignment in most LLM pipelines.

### 14.5 The Preference-Optimization Family

| Method | Year | Idea | Pros / cons |
|---|---|---|---|
| **RLHF (PPO)** | 2022 (InstructGPT) | Reward model + PPO | Most control, online; complex, unstable |
| **DPO** | 2023 | Implicit reward in a classification loss | Simple, stable; offline only |
| **IPO** | Azar et al. 2023 | Fixes DPO's overfitting to preference gaps | More robust to noisy preferences |
| **KTO** | Ethayarajh et al. 2024 | Optimize on *binary* good/bad feedback (no pairs needed) | Works with unpaired thumbs-up/down data |
| **GRPO** | Shao et al. 2024 / DeepSeek-R1 2025 | PPO without a critic — group-relative baselines | Cheap, stable at scale; the reasoning-model method |

### 14.6 GRPO and RL for Reasoning

**GRPO — Group Relative Policy Optimization** (introduced with DeepSeekMath 2024, popularized by DeepSeek-R1, Jan 2025) replaces PPO's value-network critic with a **group-relative baseline**: for each prompt, sample a *group* of responses, score them (by reward model or verifier), and compute advantages as each response's score relative to the group mean:

```
A_i = (r_i − mean(r_group)) / std(r_group)     for each response in the group
```

No critic network → no value-function training instability, less memory, and the same policy-gradient machinery. This made large-scale RL for *reasoning* tractable. The reasoning-RL recipe (R1-style):

- **Verifiable rewards (RLVR — Reinforcement Learning with Verifiable Rewards)**: reward = correctness against a ground-truth checker — math answers, code tests — instead of a learned reward model. Objective, cheap, not gameable.
- **Outcome reward models (ORM)**: learned scorers for tasks without automatic checks, judging the final answer.
- The result: models that *learn to think* — chain-of-thought emerges from RL pressure, as in DeepSeek-R1 and OpenAI's o-series.

GRPO + RLVR is now the standard for post-training reasoning models and is steadily displacing reward-model RLHF for fact-grounded domains (math, code, structured finance analytics).

### 14.7 Connection to This Repo's LLM Series

RLHF/DPO/GRPO are the **training-time alignment algorithms** in this repo's LLM story. `knowledge_distillation_guide.md` covers the complementary compression/teacher-student axis; `beyond_rag_guide.md` covers self-improving systems and drift that matter once aligned models run in production; `hybrid_multi_agent_systems_guide.md` covers the multi-agent orchestration these aligned models are embedded in; and `llm_development_risks_security_guide.md` covers the evaluation and explainability concerns that apply to RL-trained policies (Section 16.7). For a banking architect: RLHF/DPO/GRPO are exactly the techniques a bank's LLM-alignment team (or vendor) will claim to use for a copilot — this section is the vocabulary for judging the claim.

## 15. Multi-Agent Reinforcement Learning

### 15.1 The Multi-Agent Setting

**MARL** studies environments with *multiple learning agents*: cooperative (agents share a reward — trading desk teamwork), competitive (opposed rewards — market making vs the market, adversarial fraud), or mixed (negotiation, auctions). The MDP generalizes to a **Markov game** (stochastic game): each agent has its own action and often its own reward, and the transition depends on *all* agents' actions.

### 15.2 Independent Learning vs CTDE

- **Independent learning** (IQL, IPPO): each agent trains its own single-agent algorithm, treating others as part of the environment. Simple, but suffers **non-stationarity** — the environment changes as other agents learn, breaking the convergence assumptions.
- **CTDE — Centralized Training, Decentralized Execution**: train with global information (all agents' observations/actions — critics see everything), execute with local information only (each agent's policy sees only its own observation). The standard modern paradigm: critics can be centralized because they're discarded at execution time.

### 15.3 Key MARL Algorithms

| Algorithm | Year | Mechanism |
|---|---|---|
| **VDN** | Sunehag et al. 2018 | Value decomposition: team Q = sum of per-agent Q's (cooperative) |
| **QMIX** | Rashid et al. 2018 | Monotonic mixing network combines per-agent utilities into a joint Q, preserving argmax consistency |
| **MADDPG** | Lowe et al. 2017 | DDPG with centralized critics (one per agent, seeing all obs/actions) for mixed cooperative-competitive games |
| **IPPO / MAPPO** | 2021 | PPO applied per-agent / with centralized value functions — surprisingly strong baselines in cooperative settings |

### 15.4 MARL Challenges

- **Non-stationarity**: other agents' learning changes the transition dynamics from any one agent's viewpoint — the environment is a moving target.
- **Credit assignment across agents**: which agent caused the team outcome? Value-decomposition methods (VDN/QMIX) exist precisely to answer this.
- **Exploration and coordination**: agents must learn to coordinate on shared conventions (who covers which client segment, who quotes the spread).
- **Scalability**: joint action spaces grow exponentially — function approximation and factored representations are mandatory.

For banking context: a multi-agent view of the market (our trader vs other banks' traders, or a suite of product-pricing agents sharing a customer base) is closer to reality than a single-agent one — see `hybrid_multi_agent_systems_guide.md` for the agent-team orchestration lens, and Section 16.8 for why banks still start with single-agent/bandit formulations.

## 16. RL in Finance and Banking

### 16.1 Why Finance Is a Natural RL Domain

Finance is sequential decision-making under uncertainty with delayed, monetary rewards — the MDP abstraction fits almost eerily well: state = market/customer/portfolio features; action = trade, price, limit, offer; reward = P&L, fee income, risk-adjusted return; discount γ = cost of capital / horizon. Every major banking use case maps onto the framework:

| Banking use case | State | Action | Reward |
|---|---|---|---|
| Algorithmic trading / execution | Order book, positions, volatility | Order type, size, timing | Execution P&L, slippage |
| Market making | Book imbalance, inventory | Quoted bid/ask | Spread capture − inventory risk |
| Portfolio optimization | Returns, risk factors, holdings | Weight vector (continuous) | Risk-adjusted return |
| Dynamic pricing (deposits/loans/fees) | Rates, balances, competition, elasticity | Price/rate offer | Net interest margin, fee income |
| Credit limit setting | Customer features, usage, risk score | Limit change | Interest − expected loss |
| Fraud screening | Transaction features, device, history | Pass / review / block | Loss avoided − friction cost |
| Collections strategy | Account state, contact history | Channel, timing, offer | Recovered amount − cost |
| Customer journey / offers | Profile, behavior, channel | Offer/promo | Uplift in engagement/revenue |
| Hedging | Portfolio greeks, market state | Hedge rebalance | Risk reduction − transaction cost |

### 16.2 Algorithmic Trading and Execution

**Trade execution** (splitting a large order over time to minimize market impact) is the most-studied RL trading problem: DQN/PPO/SAC agents learn to time order submission against order-book state. **Market making** — quoting two-sided prices and managing inventory risk — is a natural RL control problem (state: inventory + book; action: spread/offset; reward: P&L − inventory penalty). Key practical facts:

- **Execution is episodic and simulator-friendly** — a good limit-order-book simulator enables offline training before any live capital is at risk.
- **Reward design dominates**: reward = *implementation shortfall* (signed P&L vs arrival price benchmark), not raw P&L; inventory penalties encode risk appetite.
- Cross-references: `asset_backed_trading_guide.md` for the trading-product context; `low_latency_cpp_development_guide.md` for the execution stack RL would sit on top of.

### 16.3 Portfolio Optimization

Continuous action spaces (weights per asset) make **DDPG/TD3/SAC** the natural fit for dynamic asset allocation and rebalancing — the policy outputs a weight vector each period, the reward is risk-adjusted return (Sharpe-like), and transaction costs enter the reward as a penalty to stop churn. Practical notes:

- **State must include transaction-cost and liquidity features**, or the agent will learn to overtrade.
- **Benchmarks and constraints** (tracking error, notional limits) are hard constraints — encode them in the action parameterization or as reward penalties (constrained MDPs, Section 19.6).
- Research results are mixed versus classical baselines (mean-variance, risk parity); RL's edge shows in *regime-switching and cost-aware* settings where static optimizers can't adapt.

### 16.4 Dynamic Pricing

Deposit/loan pricing and fee optimization are contextual, repeated decisions with measurable feedback — a good bandit or RL fit: state = customer segment + market rates + balances; action = rate/price tier; reward = margin contribution. **Contextual bandits** (Section 18) are the entry point; full RL (with customer balance dynamics as the state transition) is the advanced version. Pricing RL must respect: fairness/redlining constraints, rate-regulation disclosure, and the fact that *prices affect retention over long horizons* (the delayed reward that justifies RL over bandits).

### 16.5 Credit Decisions, Fraud, and Collections

- **Credit limit setting** — dynamic limits are sequential decisions (raise/lower/hold based on usage and risk drift); see `banking_limits_domain_guide.md` for the domain model. Offline RL from historical limit/usage/outcome logs is the realistic route.
- **Fraud detection** — adaptive screening is a *cost-sensitive* decision (block vs friction vs loss). Contextual bandits choose the review action per transaction; see `financial_fraud_detection_at_scale_guide.md`.
- **Collections/recovery** — choose channel, timing, and offer per account state to maximize recovery net of cost; episodic, logged data rich, and reward is directly measurable — one of the most RL-ready banking processes.

### 16.6 Customer Journey and Risk Management

- **Customer journey**: bandits for offer/personalization optimization (which product, which channel, which incentive) — the "adaptive A/B test" pattern; see `sales_methodology_frameworks_guide.md` for the sales-side context.
- **Hedging**: RL for dynamic hedging learns rebalance policies under transaction costs and model risk; the state is greeks + market features; the reward is P&L minus hedging cost and risk penalty; see `financial_risk_compliance_systems_guide.md`.

### 16.7 The Banking Reality Check

Production RL in banking is **rare**, and the reasons are structural — this is the most important subsection for a banking architect:

| Barrier | Why it bites | Mitigation |
|---|---|---|
| **Regulation / model governance** | RL policies are models; SR 11-7 (and BCBS 239-adjacent expectations) require validation, monitoring, and *challenger* processes for models that drive decisions | Treat policies as governed models; document MDP design; validate in simulation; offline RL keeps the training process auditable |
| **Explainability** | Policies are opaque (neural nets); regulators and front office demand to know *why* a decision was made — see the explainability discussion in `llm_development_risks_security_guide.md` | Prefer interpretable states/features; use bandits/linear policies where possible; post-hoc explainers (SHAP on Q/policy) |
| **No online exploration** | You cannot trial-and-error with real money, real clients, or real risk limits | Offline RL on logged data; simulators; bandits with tight explore budgets |
| **Non-stationarity** | Markets, rates, and customer behavior drift — a policy trained on 2023 data is stale by 2026 | Retraining pipelines, drift detection (`drift_detection_methods_guide.md`), regime features |
| **Safety** | Exploration *is* risky action; a bad policy update can lose money or breach limits | Constrained MDPs, action clipping, hard-rule overrides, shadow deployment |
| **Evaluation** | No ground truth for "what would have happened if we'd acted differently" | Off-policy evaluation (Section 19.3), backtesting on historical data, shadow mode |

### 16.8 What Banks Actually Do Today

The pragmatic 2026 pattern in banking AI teams: **bandits for low-stakes adaptive decisions** (offers, screening thresholds, pricing experiments), **offline RL in simulation-first research** (execution, collections) with heavy backtesting, and **RLHF-style alignment for internal LLM copilots** — with full online RL trading portfolios still confined to hedge-fund research desks and a handful of market-making franchises. The bank-friendly path is: logged data → offline RL → simulator validation → shadow mode → narrow, guardrailed deployment. Sections 17 and 18 are the toolkits for exactly that path.

## 17. Offline RL: The Answer for Banking

### 17.1 The Offline Setting

**Offline RL (batch RL)** learns a policy from a *fixed dataset of logged transitions* — no exploration, no environment interaction, ever. This is precisely the banking data regime: years of historical trades, limit decisions, pricing offers, and collections actions, with their outcomes. The catch: the data was generated by *some other* (usually conservative) policy, and standard off-policy algorithms **overestimate the value of actions they've never seen** — a naive Q-learning on logged data will confidently recommend actions that were never tried and whose true value is unknown.

### 17.2 The Core Offline Problem: Distribution Shift

The value of an action not present (or rare) in the dataset is extrapolation, not estimation. Q-learning's `max` then fires on these extrapolated values — **the overestimation problem, amplified**. Offline RL methods exist to suppress exactly this. Their design axis: *how pessimistic should the learner be about unseen actions?*

### 17.3 The Offline RL Family

| Method | Year | Mechanism | Philosophy |
|---|---|---|---|
| **BCQ** (Batch-Constrained Q-learning) | Fujimoto et al. 2019 | Constrain actions to a generative model of the *data's* actions — only query Q for actions the behavior policy plausibly took | Stay near the data |
| **CQL** (Conservative Q-learning) | Kumar et al. 2020 | Add a regularizer that *pushes down* Q on actions not in the data: `min_Q [ α·E_{μ(a|s)}[Q] − E_data[Q] ] + Bellman error` — pessimistic value estimation | Penalize the unknown |
| **IQL** (Implicit Q-learning) | Kostrikov et al. 2022 | Learn only the *upper expectile* of the value distribution; no explicit policy constraint, no function-approx instability — robust and simple | Trust the data's good actions |
| **Decision Transformer (DT)** | Chen et al. 2021 | Reframe RL as *sequence modeling*: a transformer predicts the next action given (return-to-go, state, action) history — supervised training on trajectories | RL as supervised learning |
| **CQL→SAC/TD3 adapters** | — | Standard actor-critic algorithms with pessimism/constraints added | Offline versions of online methods |

### 17.4 CQL in More Depth

CQL is the current default for many offline problems. Its loss adds a term that **minimizes Q under the policy's action distribution while maximizing Q under the data distribution** — i.e., "be pessimistic about what we haven't seen, optimistic about what we have." This single change turns an unstable offline Q-learning into a usable policy learner and — critically for banking — the *pessimism is tunable* (`α`), so a risk committee can dial conservatism up or down.

### 17.5 Decision Transformer: RL as Sequence Modeling

**DT** is the most conceptually striking offline method: it discards the policy-gradient/Bellman machinery entirely and trains a **transformer to predict the next action** from a context window of `(return-to-go, state, action)` triples. At deployment, you feed the *desired* return-to-go and the transformer generates a trajectory achieving it. DT's significance: (1) it turns RL into stable, scalable supervised training; (2) it handles long horizons and multi-task datasets gracefully; (3) it kicked off a research trend (Trajectory Transformer, Q-Transformer) that continues into 2026.

### 17.6 Offline RL for Banking

The fit is unusually good:

- **Safe**: no exploration risk — the policy is learned and validated entirely on historical data.
- **Regulatory-friendly**: the training process is a documented batch job; the dataset is auditable; SR 11-7 model risk management applies cleanly (unlike online RL's continuous adaptation).
- **Evaluatable**: backtest on held-out historical periods; off-policy evaluation gives principled estimates before any deployment.
- **Applicable across the bank**: execution policies from historical order data, credit-limit policies from limit/usage/outcome logs (`banking_limits_domain_guide.md`), collections strategies from recovery histories, pricing from offer/response logs.

Caveats: offline RL inherits all the *dataset's* biases (a period without a crisis teaches nothing about crises), needs careful reward re-construction from logs (often the hardest part — outcomes must be re-derived, e.g., attributing P&L and costs), and standard offline benchmarks are toy problems — expect real-world gaps. Still, for a regulated bank, offline RL (CQL/IQL/DT) is the current practical frontier — the only flavor of deep RL that fits the governance and safety constraints of Section 16.7.
## 18. Contextual Bandits: The Single-Step RL

### 18.1 Multi-Armed Bandits

The **multi-armed bandit** is RL stripped to its essence: `N` actions ("arms"), each with an unknown reward distribution, and the agent must maximize total reward over a sequence of pulls. There is **no state, no transition, no horizon** — every decision is independent. The entire problem is the exploration/exploitation trade-off of Section 3, and it is the right model for *repeated single-step decisions*: which ad to show, which offer to make, which screening threshold to apply.

The classical algorithms:

| Algorithm | Mechanism | Notes |
|---|---|---|
| ε-greedy | Random arm with probability ε | Simple baseline; wasteful exploration |
| **UCB1** | `argmax_a [ μ̂_a + √(2·ln N / n_a) ]` | Optimism bonus; provably near-optimal regret |
| **Thompson sampling** | Sample each arm's mean from its posterior (Beta after Bernoulli rewards); pull the max sampled | Bayesian; best practical regret; trivially extensible |
| Exp3 | Exponential-weight exploration for adversarial rewards | For non-stationary/adversarial settings |

### 18.2 Contextual Bandits

The **contextual bandit** adds features: at each step the agent observes context `x` (customer profile, market state, transaction features), picks an action, and receives a reward. The agent must learn *which action works for which context* — the mapping from features to actions is the policy. This is **an MDP with horizon 1**: state = context, no transition. It is the production workhorse of adaptive decision-making:

| Method | Idea |
|---|---|
| **LinUCB** (Li et al., 2010) | Assume linear expected reward `E[r|x,a] = θ_aᵀx`; UCB-style confidence ellipsoids per arm; closed-form ridge-regression updates |
| **Thompson sampling (contextual)** | Bayesian linear/logistic model per arm (or a neural net); sample parameters, pick the best sampled reward |
| Neural bandits | Deep network + uncertainty estimates (dropout ensembles, bootstrapped nets) — for rich contexts |

### 18.3 Production Applications

- **Recommendation and offers**: which product/channel/incentive for which customer — the bank's customer-journey optimizer (Section 16.6).
- **Pricing**: which price tier for which segment, adapting to response in near real time.
- **Fraud screening**: which review action (pass / step-up / block) for which transaction context, balancing loss vs friction (`financial_fraud_detection_at_scale_guide.md`).
- **Bandits as adaptive A/B tests**: instead of a fixed 50/50 split, a bandit continuously shifts traffic toward the winning variant — statistically valid (with proper safeguards) and much faster to deploy. This is the most common "RL" a bank actually runs in production.

### 18.4 From Bandits to Full RL

Bandits are the bridge into RL: they share the exploration/exploitation machinery, off-policy evaluation (Section 19.3), and Thompson sampling heritage, but have **no temporal dimension**. The upgrade path is principled: bandit → contextual bandit → MDP with short horizon → full RL. When retention, balance dynamics, or state carry-over matter (pricing that affects future balances; limits that affect future usage), the bandit's assumption breaks and full RL earns its complexity. The practical rule: **start with a bandit, add state only when the problem demands it.**

## 19. Evaluation and Practical Considerations

### 19.1 Evaluating RL Agents

RL evaluation is harder than supervised evaluation: there is no fixed test set — the *policy's own actions* generate the evaluation distribution. Standard toolkit:

| Method | What it does | Use for |
|---|---|---|
| **Mean return over episodes** | Average discounted return across many evaluation rollouts (fixed seed set) | Headline metric |
| **Learning curves** | Return vs environment steps (with multiple seeds; shaded intervals) | Diagnosing stability/convergence |
| **Multiple seeds** | RL is seed-sensitive; report median + spread over ≥5 seeds | Honest comparisons |
| **Off-policy evaluation (OPE)** | Estimate a target policy's value from logged data without deploying it (importance sampling, doubly-robust, FQE) | Banking: choosing policies from historical logs without live risk |
| **Robustness tests** | Perturb dynamics/rewards/initial states; check policy degradation | Deployment confidence |

### 19.2 Environment Design: The Key Practical Problem

The environment (simulator) *is* the RL project's foundation — and its most underestimated cost. For banking: a **market simulator** (limit-order-book replay, agent-based market models) or a **banking environment** (customer behavior, collections, pricing response models). Rules of thumb:

- The simulator's fidelity determines what the policy learns — *a policy trained on a wrong simulator is confidently wrong*.
- Start simple (bandit/linear models), then add fidelity where it changes decisions.
- **Sim-to-real gap** (simulator vs production reality) must be measured: hold out real logs, compare behavior distributions.
- Reuse where possible — market replay from historical data is far cheaper and more credible than generative market simulators for evaluation (even when a generative model trains the policy).

### 19.3 Reward Design

- **Sparse vs dense**: sparse rewards (win/lose) are hard to learn from; dense rewards (progress signals) accelerate learning but invite gaming. Reward shaping (adding intermediate signals) must preserve the optimal policy (potential-based shaping does).
- **Reward hacking**: the agent exploits loopholes — maximizing the *proxy* reward, not the intended objective (e.g., a collections agent learning to harass customers to hit recovery targets, or an execution agent gaming the benchmark). Symptoms: high learned reward, bad real outcomes. Mitigations: careful reward audit, constraints, KL-style penalties, monitoring.
- In banking, *every* reward is a business metric (P&L, NIM, recovery) with side effects — reward design is a business decision, not a coding detail. Document it like a model specification (SR 11-7 context, Section 16.7).

### 19.4 Training Stability and Reproducibility

RL is notoriously hyperparameter-sensitive: small changes in learning rate, batch size, γ, λ, clip range, or network width flip results. Practical hygiene:

- Fix seeds and report multi-seed statistics; pin library versions; log everything (config, env version, seeds) — reproducibility is a governance requirement in banking, not an academic nicety.
- Tune with small budgets first; use the library defaults as starting points (Stable-Baselines3 defaults are well-tested).
- Watch the *deadly triad* for value methods (Section 10.7) and entropy collapse for policy methods (Section 11.3) as the first failure diagnoses.

### 19.5 Monitoring, Drift, and Safety

- **Policy drift / distribution shift**: a deployed policy faces data that differs from training (market regimes, client mix). Monitor state-action distribution shift and performance; see the self-improving-systems discussion in `beyond_rag_guide.md` and `drift_detection_methods_guide.md`.
- **Safe RL / constrained MDPs**: add constraints (`E[cost] ≤ budget`) via Lagrangian methods or constrained policy optimization; risk-sensitive RL optimizes risk-adjusted objectives (CVaR, entropic risk) — directly relevant to banking limits (`banking_limits_domain_guide.md`).
- **Guardrails**: hard-rule overrides (position limits, price bands), shadow deployment before live, kill switches. Exploration budgets must be approved like risk limits.

### 19.6 Compute

Deep RL is compute-heavy in a distinctive way: it needs *parallel environment rollouts* (CPU/vectorized envs) plus GPU training, and wall-clock is dominated by data collection. Practical scaling: vectorized environments (Gymnasium's `VectorEnv`, Sample-Factory's async rollouts), 1–8 GPUs for typical PPO/SAC training, and much more for RLHF-scale runs (thousands of GPUs for frontier-model RL — GRPO's critic-free design is partly a *compute* optimization). Cloud spot instances and env-vectorization libraries are the standard cost levers.

### 19.7 Libraries

| Library | What it is | Strengths | Best for |
|---|---|---|---|
| **Stable-Baselines3 (SB3)** | The standard PyTorch RL library | Clean, well-tested PPO/SAC/TD3/DQN/A2C; Gymnasium integration; simple API | Production-ish single-agent RL, learning, banking prototypes |
| **RLlib (Ray)** | Distributed RL on Ray | Scalable, multi-agent support, custom policies, tuned experiments | Distributed training, MARL, large-scale experiments |
| **CleanRL** | Single-file implementations | Readable, educational, reproduction-friendly | Learning internals, research baselines |
| **Dopamine** | Google's research framework | DQN-family focus (Rainbow, IQN) | Value-based research |
| **Acme** | DeepMind's research library | Modular, scalable, distributed (JAX/TF) | Research, distributed RL |
| **Gymnasium** | The environment standard (successor to Gym) | Hundreds of envs, vectorized API, env design tools | Environments/simulators for everything else |
| **Sample-Factory** | High-throughput training | Async rollouts, very fast wall-clock | Large-scale PPO-style training |
| **Tianshou** | Lightweight PyTorch RL | Clean, fast, many algorithms | Research and education |

Practical guidance for a bank team: **Gymnasium for environments + Stable-Baselines3 for algorithms** covers 80% of needs; RLlib when distribution or multi-agent enters; CleanRL when you must understand exactly what a PPO update does (which you will, for model governance).

## 20. When to Use RL: The Decision Framework

### 20.1 The Fits / Overkill Test

| RL fits when… | RL is overkill when… |
|---|---|
| **Sequential decisions** — actions now affect future states | **Single-step decisions** — use bandits or supervised classification |
| **Delayed rewards** — feedback arrives after several steps | **Immediate feedback** — supervised learning suffices |
| **Interaction available** — a simulator or live env exists (or **logged data** for offline RL) | **No interaction possible** — use offline RL on logs, or supervised behavior cloning |
| **The policy must adapt online** — environment/behavior drifts | **Stationary mapping** — a trained classifier/optimizer is enough |
| The optimal action is not derivable by closed-form optimization | A closed-form optimizer exists (e.g., mean-variance with known parameters) |

### 20.2 The Decision Flow

```
Is the decision sequential (actions affect future states)?
├─ No  → Single-step: contextual bandit (or supervised if no exploration needed)
└─ Yes → Is there delayed reward?
         ├─ No  → Bandit / supervised (treat as one-step per context)
         └─ Yes → Can you interact (simulator / live, safely)?
                  ├─ No  → OFFLINE RL on logged data (CQL/IQL/DT) — the banking default
                  └─ Yes → Action space?
                           ├─ Discrete → DQN family / PPO
                           └─ Continuous → SAC or TD3 (off-policy) / PPO (on-policy)
                           Then ask: sample efficiency critical?
                           └─ Yes → model-based (MuZero/Dreamer) or offline RL
```

### 20.3 Pragmatic Guidance

- **Simple adaptive decisions** (offers, screening thresholds, pricing experiments): start with a **contextual bandit** — Thompson sampling or LinUCB. Deployable today, governable, and a natural stepping stone.
- **Continuous control with a simulator** (execution, allocation research): **PPO** (on-policy, robust) or **SAC** (off-policy, sample-efficient). TD3 when a deterministic policy is preferred.
- **Banking data without interaction** (credit limits, collections, execution from logs): **offline RL** — CQL or IQL first, Decision Transformer when sequence structure dominates.
- **LLM alignment**: **DPO** for most preference tuning; **GRPO + RLVR** for reasoning/verifiable-reward domains; full **PPO-RLHF** when online reward-model optimization is worth the complexity.
- **When in doubt, benchmark against the incumbent**: RL should beat the current policy (rule-based, supervised, or static) in simulation *and* on out-of-sample logs before it earns a pilot. This is both good engineering and the SR 11-7 argument.

## 21. The Algorithm Cheat-Sheet

| Algorithm | Type | Action space | Sample efficiency | Stability | Best for | Year / origin |
|---|---|---|---|---|---|---|
| Policy iteration / value iteration | Value-based (DP) | Discrete | — (needs model) | High (guaranteed) | Small MDPs, planning, theory | Bellman 1957 / Howard 1960 |
| Monte Carlo | Value-based | Discrete | Low | Medium (high variance) | Episodic prediction | 1980s |
| TD(0) | Value-based | Discrete | Medium | Medium | Online prediction | Sutton 1988 |
| SARSA | Value-based, on-policy | Discrete | Medium | High (conservative) | Risk-averse control | Rummery & Niranjan 1994 |
| Q-learning | Value-based, off-policy | Discrete | Medium-high | Medium | Discrete control, replay | Watkins 1989 |
| DQN | Deep value-based, off-policy | Discrete | Medium | Medium (needs tricks) | Atari/games, discrete decisions | Mnih et al. 2015 |
| Rainbow | Deep value-based, off-policy | Discrete | Medium-high | Medium-high | Best discrete deep RL | Hessel et al. 2017 |
| C51 / QR-DQN / IQN | Distributional value-based | Discrete | Medium-high | Medium | Risk-sensitive discrete RL | 2017–2018 |
| REINFORCE | Policy-based, on-policy | Both | Low | Low (high variance) | Simple PG, theory | Williams 1992 |
| A2C / A3C | Actor-critic, on-policy | Both | Low-medium | Medium | Parallel on-policy training | Mnih et al. 2016 |
| TRPO | Actor-critic, on-policy | Both | Low-medium | High (hard KL) | Stable PG with theory | Schulman et al. 2015 |
| PPO | Actor-critic, on-policy | Both | Low-medium | High | Default on-policy; RLHF | Schulman et al. 2017 |
| DDPG | Actor-critic, off-policy | Continuous | Medium | Low-medium (fragile) | Continuous control (legacy) | Lillicrap et al. 2016 |
| TD3 | Actor-critic, off-policy | Continuous | Medium | High | Robust deterministic control | Fujimoto et al. 2018 |
| SAC | Actor-critic, off-policy, max-entropy | Continuous | High | High | Default continuous control | Haarnoja et al. 2018 |
| GAE | Advantage estimator | — | — | — | Advantage estimation for PG | Schulman et al. 2016 |
| Dyna-Q | Model-based, value-based | Discrete | High (imagined data) | Medium | Model + value hybrid | Sutton 1990 |
| MuZero | Model-based + MCTS | Discrete (cont. variants) | Very high | High | Sample-efficient planning | Schrittwieser et al. 2020 |
| Dreamer | Model-based (latent), actor-critic | Continuous | Very high | Medium | Visual/continuous control | Hafner et al. 2020–2023 |
| RLHF (PPO) | Preference + RL, on-policy | Discrete (tokens) | Medium | Medium | LLM alignment | InstructGPT 2022 |
| DPO | Preference optimization | Discrete (tokens) | High (offline) | High | LLM alignment default | Rafailov et al. 2023 |
| GRPO | Policy-based, on-policy, no critic | Discrete (tokens) | Medium | High | Reasoning-model RL (R1) | Shao et al. 2024 / DeepSeek-R1 2025 |
| BCQ | Offline value-based | Discrete/continuous | High (offline) | Medium | Offline RL, safe policies | Fujimoto et al. 2019 |
| CQL | Offline value-based (pessimistic) | Discrete/continuous | High (offline) | Medium-high | Offline RL default; banking | Kumar et al. 2020 |
| IQL | Offline (implicit Q) | Discrete/continuous | High (offline) | High | Robust simple offline RL | Kostrikov et al. 2022 |
| Decision Transformer | Offline sequence model | Both | High (offline) | High (supervised) | RL-as-sequence-modeling | Chen et al. 2021 |
| UCB / Thompson / LinUCB | Bandit | Discrete (contextual: discrete arms) | Very high | High | Adaptive single-step decisions | 1933 / 2010 |

## 22. The Future: 2026 and Beyond

| Trend | What it is | Why it matters for banking |
|---|---|---|
| **RLHF/GRPO dominating LLM alignment** | Reasoning models trained with verifiable-reward RL (R1-style) become the default for math/code/structured tasks | Banks' copilots and analytics will be built on reasoning-aligned models; GRPO/RLVR vocabulary becomes procurement-required |
| **World models (Genie-style)** | Generative world models trained at internet scale produce interactive, controllable environments without hand-built simulators | Cheaper, richer simulators for strategy and trading training — but *generated* data needs validation (see sim-to-real, Section 19.2) |
| **RL + GenAI for agent behavior** | RL trains LLM *agents* (tool use, browsing, multi-step workflows) — behavior RL, not just text alignment | Agentic banking workflows (`agentic_workflows_guide.md`, `hybrid_multi_agent_systems_guide.md`) get RL-trained policies for reliability |
| **Offline RL maturity** | Benchmarks, libraries, and practice harden; CQL/IQL/DT-class methods ship in production stacks | The banking adoption path: offline RL becomes the standard way to turn decision logs into policies |
| **Safe/constrained RL and regulation** | Constrained MDPs, risk-sensitive objectives, and interpretability tooling mature as AI regulation (EU AI Act risk tiers, SR 11-7-style governance) tightens | Governable RL is the only RL that gets deployed in banks; safety tooling is the enabler |
| **The RL renaissance** | RL is reframed from "game-playing research" to the *training-time engine of alignment and reasoning* — the field's center of gravity moved into LLM post-training | Every bank ML team now has RL-relevant skills on the roadmap, and RL concepts enter model-governance discussions |

The trajectory for practitioners: RL's hard-won core (MDPs, TD, policy gradients, actor-critics, offline learning) is stable and will outlive any specific algorithm; the frontier is *where* RL is applied — alignment, agents, world models, and, for banks, the safe offline exploitation of decision data.

## 23. Glossary

| Term | Definition |
|---|---|
| **MDP** | Markov Decision Process — the formal model `(S, A, P, R, γ)` of sequential decision-making |
| **Policy** | The agent's decision rule `π(a|s)` — mapping from states to actions (deterministic or stochastic) |
| **Return** | Cumulative discounted reward `G_t = Σ γ^k r_{t+k}` |
| **Discount factor γ** | How much future reward is worth relative to present reward (0 to 1) |
| **Horizon** | Number of decision steps (finite/episodic, infinite, or discounted-infinite) |
| **State-value V(s)** | Expected return from state `s` under a policy |
| **Action-value Q(s,a)** | Expected return from taking action `a` in state `s`, then following a policy |
| **Advantage A(s,a)** | `Q(s,a) − V(s)` — how much better an action is than the state's average |
| **Bellman equation** | Recursive consistency equation relating a state's value to successors' values; basis of all TD methods |
| **Bootstrapping** | Updating an estimate using another estimate (the next state's value) — TD's defining mechanism |
| **TD error** | `δ = r + γV(s') − V(s)` — the prediction-correction signal of TD learning |
| **TD / TD(λ)** | Temporal Difference learning; λ interpolates between 1-step (TD(0)) and Monte Carlo |
| **Eligibility trace** | Decaying memory of recently visited states/actions enabling λ-return updates |
| **SARSA** | On-policy TD control using the actually-taken next action |
| **Q-learning** | Off-policy TD control using the greedy next action (Watkins 1989) |
| **Experience replay** | Buffer of past transitions sampled for training; breaks correlation, reuses data |
| **Target network** | Slowly-updated copy of the Q-network used to compute stable targets |
| **Deadly triad** | Instability from combining function approximation + bootstrapping + off-policy |
| **Policy gradient** | Gradient of expected return w.r.t. policy parameters: `∇log π(a|s)·A(s,a)` |
| **REINFORCE** | Monte Carlo policy gradient using full returns (Williams 1992) |
| **Actor-critic** | Architecture with a policy (actor) and value function (critic/baseline) |
| **GAE** | Generalized Advantage Estimation — λ-weighted advantage estimator (Schulman 2016) |
| **Entropy** | `H(π) = −Σ π log π` — randomness of the policy; entropy bonuses encourage exploration |
| **Trust region** | Constraint keeping policy updates close in KL/ratio space (TRPO, PPO clipping) |
| **Surrogate objective** | Approximation of the return optimized w.r.t. the new policy (PPO's clipped objective) |
| **On-policy / off-policy** | Learning from current-policy data only / from any data (e.g., replay buffer) |
| **Model-based RL** | Learning/using an explicit dynamics model and planning inside it |
| **MCTS** | Monte Carlo Tree Search — lookahead planning used by AlphaZero/MuZero |
| **Reward model** | Learned function scoring outputs (e.g., from human preferences via Bradley-Terry) |
| **Bradley-Terry model** | Pairwise preference probability `σ(R(y_w) − R(y_l))` used to train reward models |
| **RLHF** | Reinforcement Learning from Human Feedback — SFT → reward model → PPO pipeline |
| **DPO** | Direct Preference Optimization — policy optimized directly on preference pairs |
| **GRPO** | Group Relative Policy Optimization — PPO without a critic, group-relative baselines |
| **RLVR** | Reinforcement Learning with Verifiable Rewards — objective correctness signals (math, code) |
| **Offline RL** | Learning from a fixed logged dataset without environment interaction |
| **CQL / IQL / BCQ** | Conservative / Implicit / Batch-Constrained offline Q-learning methods |
| **Decision Transformer** | Offline RL reframed as supervised sequence prediction |
| **Bandit** | Horizon-1 RL: repeated single-step decisions with exploration/exploitation |
| **Contextual bandit** | Bandit with per-step features/context (LinUCB, contextual Thompson sampling) |
| **UCB** | Upper Confidence Bound — optimistic exploration by uncertainty bonus |
| **Thompson sampling** | Bayesian exploration by sampling from posteriors |
| **CTDE** | Centralized Training, Decentralized Execution — MARL paradigm |
| **Non-stationarity** | Environment dynamics changing over time (other agents learning, market drift) |
| **OPE** | Off-policy evaluation — estimating a policy's value from logged data |
| **Reward hacking** | The policy optimizing the proxy reward while violating the intended objective |
| **Constrained MDP** | MDP with cost constraints; basis of safe RL |

## 24. References and Further Reading

### 24.1 Canonical Papers

- Sutton & Barto, *Reinforcement Learning: An Introduction* (2nd ed., 2018) — the field's textbook; free online.
- Watkins, *Q-learning* (1989); Sutton, *Learning to predict by the methods of temporal differences* (1988); Williams, *Simple statistical gradient-following algorithms for connectionist reinforcement learning* (REINFORCE, 1992).
- Mnih et al., *Human-level control through deep reinforcement learning* (DQN, Nature 2015); van Hasselt et al., *Deep RL with Double Q-learning* (2016); Wang et al., *Dueling Network Architectures* (2016); Schaul et al., *Prioritized Experience Replay* (2016); Hessel et al., *Rainbow* (2017).
- Schulman et al., *TRPO* (2015); *High-Dimensional Continuous Control Using GAE* (2016); *PPO* (2017).
- Mnih et al., *Asynchronous Methods for Deep RL* (A3C, 2016); Lillicrap et al., *DDPG* (2016); Fujimoto et al., *TD3* (2018); Haarnoja et al., *SAC* (2018).
- Silver et al., *AlphaGo* (2016) / *AlphaZero* (2018); Schrittwieser et al., *MuZero* (2020); Hafner et al., *Dreamer* (2020–2023); Ha & Schmidhuber, *World Models* (2018).
- Ouyang et al., *Training language models to follow instructions with human feedback* (InstructGPT/RLHF, 2022); Rafailov et al., *DPO* (2023); Shao et al., *DeepSeekMath: GRPO* (2024); DeepSeek-AI, *DeepSeek-R1* (2025).
- Kumar et al., *CQL* (2020); Kostrikov et al., *IQL* (2022); Chen et al., *Decision Transformer* (2021); Fujimoto et al., *BCQ* (2019).
- Lowe et al., *MADDPG* (2017); Rashid et al., *QMIX* (2018); Sunehag et al., *VDN* (2018).
- Li et al., *A Contextual-Bandit Approach to Personalized News Article Recommendation* (LinUCB, 2010).

### 24.2 Sibling Guides in This Repo

- `asset_backed_trading_guide.md` — trading products RL agents would act on
- `banking_limits_domain_guide.md` — the domain model for RL-driven credit limits
- `financial_fraud_detection_at_scale_guide.md` — adaptive fraud screening with bandits
- `financial_risk_compliance_systems_guide.md` — hedging and risk systems context
- `llm_development_risks_security_guide.md` — explainability and risk for ML/LLM systems
- `knowledge_distillation_guide.md`, `llm_model_pruning_guide.md` — model compression counterparts
- `beyond_rag_guide.md` — self-improving systems, drift, production LLM concerns
- `hybrid_multi_agent_systems_guide.md` — multi-agent orchestration and agent teams
- `agentic_workflows_guide.md` — RL-trained agent behavior in workflows
- `drift_detection_methods_guide.md` — monitoring distribution shift in deployed policies
- `low_latency_cpp_development_guide.md` — the execution stack for trading RL
- `sales_methodology_frameworks_guide.md` — customer-journey and offer optimization context

### 24.3 Learning Resources

- Spinning Up in Deep RL (OpenAI) — concise tutorial + implementations of the modern algorithms.
- Hugging Face Deep RL Course — hands-on, Gymnasium-based, free.
- CleanRL — readable single-file implementations of every major algorithm.
- Stable-Baselines3 documentation — the practical API reference.
- Sutton & Barto, *Reinforcement Learning: An Introduction* — the definitive book.

---

*Product, algorithm, and library facts as of August 2026; verify against the primary papers and official documentation before architecture decisions. This guide is part of the research repo's AI/ML series and is intended as a technical reference, not investment or compliance advice.*



