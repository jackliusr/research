# Multi-Armed Bandits: A Comprehensive Guide

**Sequential Decision-Making Under Uncertainty — Exploration vs. Exploitation**

---

## 1. Overview & Origin

The **Multi-Armed Bandit (MAB)** problem is a foundational framework in sequential decision-making under uncertainty. The name comes from casino slot machines ("one-armed bandits") — imagine a gambler facing multiple slot machines (each an "arm"), each with a different, unknown payout distribution. The gambler must decide which machines to play, in what order, and how many times, to maximize cumulative winnings.

### The Explore vs. Exploit Dilemma

Every bandit algorithm faces a fundamental tension:

- **Exploitation**: Play the arm that has given the best rewards so far (greedy choice).
- **Exploration**: Try other arms to gather information about their payoff distributions, potentially discovering a better arm.

Pure exploitation risks missing a superior arm; pure exploration sacrifices short-term reward for long-term gain. The goal is to balance both optimally.

### Formal Definition

At each time step $t = 1, 2, ..., T$:
1. The agent selects an action $A_t \in \{1, ..., K\}$ from $K$ arms.
2. The environment reveals a reward $R_t \sim \nu_{A_t}$ drawn from the chosen arm's (unknown) reward distribution.
3. The agent updates its knowledge and repeats.

The objective: **maximize expected cumulative reward** $\mathbb{E}[\sum_{t=1}^T R_t]$, or equivalently **minimize cumulative regret**.

---

## 2. Types of Bandit Problems

### 2.1 Stochastic Bandits

Each arm $k$ has a fixed but unknown reward distribution $\nu_k$ (e.g., Bernoulli with parameter $\mu_k$, or Gaussian $N(\mu_k, \sigma_k^2)$). Rewards are i.i.d. across pulls of the same arm. This is the classical setting — most introductory algorithms (epsilon-Greedy, UCB, Thompson Sampling) are designed for it.

### 2.2 Contextual Bandits

At each round, the agent observes a context vector $x_t \in \mathbb{R}^d$ (user features, item features) before choosing an arm. The reward depends on both the context and the chosen arm: $R_t = f(x_t, A_t) + \epsilon_t$. This bridges the gap between pure bandits and full reinforcement learning.

- **LinUCB**: Assumes $f$ is linear in the context features. Disjoint model (separate parameters per arm) vs. hybrid (shared + per-arm parameters).
- **Thompson Sampling with Context**: Places a Bayesian prior over the reward function's parameters $p(\theta)$, updates to posterior $p(\theta | \mathcal{D}_t)$, and samples arms proportionally to their probability of being optimal.

### 2.3 Adversarial Bandits

No statistical assumptions about rewards — an adversary chooses rewards each round, possibly based on past agent actions. The agent must still achieve sublinear regret.

- **Exp3 (Exponential-weight for Exploration and Exploitation)**: Maintains importance-weighted reward estimates and selects arms via a Gibbs (softmax) distribution. Achieves $O(\sqrt{T K \log K})$ regret against any deterministic strategy.

### 2.4 Non-Stationary Bandits

Reward distributions drift or change abruptly over time (concept drift). Algorithms must forget stale observations:

- **Sliding Window UCB**: Only considers the last $W$ observations per arm.
- **Discounted UCB**: Applies discount factor $\gamma \in (0,1)$ to older rewards, downweighting their contribution.
- **Change Detection**: Uses statistical tests (CUSUM, Page-Hinkley) to detect distribution shifts and reset arm statistics.

### 2.5 Combinatorial Bandits

At each round, the agent selects a **set** of arms (a super-arm) rather than a single arm. The reward depends on the combination chosen. Applications include: routing, network optimization, online shortest path.

### 2.6 Continuous Arms (Continuum Bandits)

The arm set is infinite and continuous (e.g., selecting a price $p \in [0,1]$). Algorithms rely on function approximation:

- **Gaussian Process Bandits**: Model the reward function as a GP. Use GP-UCB or Thompson Sampling with GP posteriors.
- **Bayesian Optimization**: Explores the continuous space using an acquisition function (Expected Improvement, Upper Confidence Bound) evaluated on a surrogate model.

---

## 3. Key Algorithms & Strategies

### 3.1 $\epsilon$-Greedy

The simplest and most widely deployed heuristic.

- With probability $\epsilon$, explore: choose a random arm uniformly.
- With probability $1 - \epsilon$, exploit: choose the arm with the highest empirical mean reward.

**Limitations**: Exploration is blind (uniform over all arms, even bad ones). Regret grows linearly $\Omega(\epsilon T)$ for fixed $\epsilon$. Decaying $\epsilon_t = 1/t$ achieves $O(\log T)$ regret but requires careful scheduling.

### 3.2 Upper Confidence Bound (UCB1)

Optimism in the face of uncertainty: select the arm with the highest plausible reward estimate.

$$A_t = \arg\max_k \left[ \hat{\mu}_k + \sqrt{\frac{2 \log t}{n_k}} \right]$$

Where $\hat{\mu}_k$ is the empirical mean of arm $k$, $n_k$ is its pull count, and $t$ is the total rounds. The confidence bonus shrinks as $n_k$ grows — encouraging exploration of under-sampled arms.

**Properties**: Achieves $O(K \log T / \Delta)$ regret (where $\Delta$ is the gap between best and second-best arm). Matches the Lai-Robbins lower bound up to constant factors.

### 3.3 Thompson Sampling (Bayesian)

A Bayesian approach: maintain a posterior distribution over each arm's reward parameter. At each round:

1. Sample $\tilde{\mu}_k \sim p(\mu_k | \text{data})$ for each arm.
2. Select $A_t = \arg\max_k \tilde{\mu}_k$.

**Why it works**: Arms are selected proportionally to their probability of being optimal. Exploration is natural and automatic — uncertain arms have wider posteriors and are sampled more frequently.

For Bernoulli rewards with Beta(1,1) prior: posterior is $Beta(\alpha_k + S_k, \beta_k + F_k)$ where $S_k$, $F_k$ are successes and failures. **State-of-the-art performance** across stochastic, contextual, and many practical settings. Near-optimal regret $O(\log T)$ with smaller constants than UCB.

### 3.4 Exp3 (Adversarial)

Maintains weight $w_k$ per arm. At round $t$: compute $p_k = (1 - \gamma) \frac{w_k}{\sum_j w_j} + \frac{\gamma}{K}$, sample $A_t \sim p$, compute importance-weighted estimate $\hat{r}_{A_t} = r_t / p_{A_t}$, update $w_k \leftarrow w_k \exp(\gamma \hat{r}_k / K)$.

**Regret**: $O(\sqrt{T K \log K})$, optimal for the adversarial setting (up to log factors).

### 3.5 LinUCB (Contextual)

Assumes expected reward is linear in $d$-dimensional features: $\mathbb{E}[r|x, a] = x^T \theta_a^*$.

- Maintains a ridge regression estimate $\hat{\theta}_a = (X_a^T X_a + I_d)^{-1} X_a^T y_a$.
- Selects $A_t = \arg\max_a \left[ x_t^T \hat{\theta}_a + \alpha \sqrt{x_t^T (X_a^T X_a + I_d)^{-1} x_t} \right]$.

**Yahoo! News result**: Li et al. (2010) demonstrated **12.5% CTR lift** over context-free bandits using disjoint LinUCB on the Yahoo! Front Page Today Module.

### 3.6 Bootstrapped Thompson Sampling

For large action spaces where exact posterior inference is intractable. Maintains $B$ bootstrapped models, each trained on resampled data. At each round: sample a bootstrap model uniformly, pick the arm it recommends. Scales to millions of arms in production recommendation systems.

### 3.7 Neural Bandits

Replaces linear models with deep neural networks for complex reward functions. Examples:

- **NeuralUCB**: Uses a neural network as feature mapping + last-layer UCB exploration.
- **Neural Thompson Sampling**: Approximate posterior over NN weights via last-layer Laplace approximation or deep ensembles.

Provides expressiveness beyond linear models at the cost of increased computational complexity.

## 4. Algorithms Comparison Table

| Algorithm | Explore Strategy | Exploit Strategy | Assumptions | Regret Bound | Comput. Complexity |
|:----------|:----------------|:-----------------|:------------|:-------------|:-------------------|
| $\epsilon$-Greedy | Random uniform $\epsilon$ | Empirical best arm | Stochastic rewards | $O(\epsilon T)$ (fixed $\epsilon$) | $O(K)$ |
| UCB1 | Confidence bound bonus | Arm with max UCB | Sub-Gaussian rewards | $O(K \log T / \Delta)$ | $O(K)$ |
| Thompson Sampling | Posterior sampling | Sample argmax | Bayesian prior | $O(\log T)$ near-optimal | $O(K \cdot \text{sample})$ |
| Exp3 | Mix uniform + weighted | Weighted softmax | Adversarial rewards | $O(\sqrt{T K \log K})$ | $O(K)$ |
| LinUCB | Contextual confidence bound | Max linear prediction | Linear reward model | $\tilde{O}(\sqrt{T})$ | $O(d^2 + dK)$ |
| NeuralBandit | NN-based uncertainty | Max NN prediction | Smooth reward function | $\tilde{O}(\sqrt{T})$ | $O(d^2_{NN} + d_{NN}K)$ |

---

## 5. Performance Metrics

### 5.1 Regret (Primary Metric)

**Cumulative regret**: $R(T) = \sum_{t=1}^T (\mu^* - \mu_{A_t})$, where $\mu^*$ is the expected reward of the optimal arm. For contextual bandits: $R(T) = \sum_{t=1}^T \mathbb{E}[r_t(a_t^*) - r_t(A_t)]$.

- **Expected regret**: $\mathbb{E}[R(T)]$ — averaged over reward randomness and algorithm stochasticity.
- **Simple regret**: $S(T) = \mu^* - \mu_{\hat{A}}$ after $T$ rounds (focuses on final arm recommendation).
- **Instance-dependent bounds**: $O(\log T)$ for stochastic bandits (depends on arm gaps $\Delta_k$).
- **Minimax bounds**: $O(\sqrt{T})$ worst-case over problem instances.

### 5.2 Total Reward / Cumulative Reward

The raw cumulative reward $\sum_{t=1}^T R_t$. Equivalent to regret up to the constant $T \cdot \mu^*$.

### 5.3 Best-Arm Identification (Pure Exploration)

After $T$ rounds, what is the probability the agent correctly identifies the best arm? Metrics include:

- **Probability of correct selection**: $P(\hat{a} = a^*)$.
- **Sample complexity**: Minimum $T$ to guarantee $\mathbb{P}(\hat{a} = a^*) \geq 1 - \delta$.

### 5.4 Sample Complexity

The number of samples (pulls) required to achieve a desired accuracy or regret threshold. Critical in clinical trials and expensive evaluation settings.

---

## 6. Mathematical Foundations

### 6.1 Regret Bounds

**Lai-Robbins Lower Bound** (1985): For any consistent policy and stochastic Bernoulli rewards, asymptotically:

$$\liminf_{T \to \infty} \frac{\mathbb{E}[R(T)]}{\log T} \geq \sum_{k: \Delta_k > 0} \frac{\Delta_k}{\text{KL}(\mu_k, \mu^*)}$$

This is the fundamental limit — no algorithm can beat $\Omega(\log T)$ regret in the stochastic setting.

**UCB1 Regret**: $R(T) \leq \sum_{k: \Delta_k > 0} \left[ \frac{8 \log T}{\Delta_k} + \left(1 + \frac{\pi^2}{3}\right) \Delta_k \right] = O(K \log T / \Delta)$.

**Thompson Sampling**: Achieves the Lai-Robbins lower bound asymptotically for Bernoulli rewards. For general exponential families, achieves $O(\sum_k \frac{\log T}{\text{KL}(\mu_k, \mu^*)})$ regret.

### 6.2 Concentration Inequalities

- **Hoeffding's Inequality**: $P(|\hat{\mu} - \mu| \geq \epsilon) \leq 2\exp(-2n\epsilon^2)$. Foundation of UCB1's confidence bound $\sqrt{\frac{2\log(1/\delta)}{n}}$.
- **Chernoff Bound**: Tighter bound for Poisson/Binomial tails. Used in variance-aware bandit algorithms.
- **Bernstein Inequality**: $P(|\hat{\mu} - \mu| \geq \epsilon) \leq 2\exp\left(-\frac{n\epsilon^2}{2\sigma^2 + 2\epsilon/3}\right)$. Enables variance-adaptive UCB algorithms with tighter exploration.

### 6.3 Bayesian vs. Frequentist Perspectives

| Aspect | Frequentist (UCB, Exp3) | Bayesian (Thompson Sampling) |
|:-------|:------------------------|:------------------------------|
| Assumption | Fixed unknown parameters | Probability distributions over parameters |
| Uncertainty | Confidence intervals (data only) | Posterior distributions (prior + data) |
| Exploration | Upper confidence bound | Random sampling from posterior |
| Regret analysis | Worst-case / instance-dependent | Bayesian expected regret |
| Prior needed | No | Yes (can be uninformative) |

### 6.4 Reward Distributions

- **Bernoulli**: $R \in \{0,1\}$, $P(R=1) = \mu$. Conjugate prior: Beta distribution.
- **Gaussian**: $R \sim N(\mu, \sigma^2)$. Conjugate prior: Normal-Inverse-Gamma.
- **Exponential Family**: Includes Poisson, Gamma, Multinomial. Enables generalized linear model (GLM) bandits with $\mathbb{E}[R|X] = g^{-1}(X^T\theta)$.

---

## 7. Applications

### 7.1 A/B Testing

Bandits offer a **30-50% reduction** in required samples compared to traditional A/B tests. They dynamically shift traffic to better variants as evidence accumulates, rather than splitting traffic 50/50 for a fixed duration and declaring a winner at the end.

### 7.2 Online Advertising

- **Ad Selection**: Choose which ad creative to show a user to maximize CTR.
- **Dynamic Creative Optimization**: Test combinations of headline, image, CTA.
- **Yahoo! News Personalization (Li et al., 2010)**: LinUCB achieved 12.5% CTR improvement, directly leading to production deployment.

### 7.3 Recommendation Systems

- **Content Discovery**: Explore new items while exploiting known user preferences.
- **Netflix/Spotify**: Bandit-based exploration for new content recommendations, cold-start item promotion.
- **News Recommendations**: Yahoo! Front Page, Google News, Microsoft Start.

### 7.4 Clinical Trials

- **Adaptive Trial Design**: Allocate more patients to treatments showing promise while maintaining statistical validity.
- **Patient-Specific Dosing**: Contextual bandits for personalized drug dosage.
- **Ethical advantage**: Fewer patients receive suboptimal treatments compared to fixed-proportion trials.

### 7.5 Hyperparameter Optimization

- **Bayesian Optimization**: A bandit over continuous hyperparameter space using Gaussian Processes.
- **Early Stopping**: Bandit algorithms (e.g., successive halving, Hyperband) decide which configurations to continue training and which to terminate.

### 7.6 Dynamic Pricing

- **Price Experimentation**: Test different price points for a product with unknown demand curve.
- **Non-stationary Bandits**: Adapt to seasonal demand shifts and competitor pricing changes.

### 7.7 Website Optimization

- Landing page A/B testing, feature rollout (canary releases), call-to-action optimization.

### 7.8 Reinforcement Learning

MAB is the **stateless RL problem** — the simplest setting where an agent learns from rewards. Contextual bandits are the bridge to full RL: they have state (context) but no state transitions. Value-function-based RL (DQN, SARSA) can be viewed as learning a contextual bandit per state.

---

## 8. MAB vs. A/B Testing: Decision Framework

| Dimension | Multi-Armed Bandits | A/B Testing |
|:----------|:--------------------|:------------|
| **Sample efficiency** | 30-50% fewer samples | Fixed 50/50 split |
| **Ethics** | Fewer users see worse variants | Half of users always see losing variant |
| **Speed** | Continuous adaptation, no fixed horizon | Fixed-duration test required |
| **Analysis complexity** | Complex — sequential analysis, stopping rules | Simple — standard hypothesis tests (t-test, chi-square) |
| **False positives** | Higher risk w/o correction | Controlled via significance level |
| **Multiple variants** | Scales well (many arms) | Requires correction (Bonferroni, FDR) |
| **Regulatory compliance** | Harder to audit / pre-register | Clean pre-registration possible |

**When to use bandits**: Many variants ($K \geq 5$), unknown time horizon, continuous optimization, many visitors, minimizing exposure to bad variants matters.

**When to use A/B testing**: Few variants (2-4), fixed horizon, strict regulatory compliance, need clean hypothesis testing, small traffic.

---

## 9. Contextual Bandits Deep Dive

### 9.1 LinUCB: Disjoint vs. Hybrid Models

**Disjoint LinUCB**: Each arm $a$ has its own parameter vector $\theta_a \in \mathbb{R}^d$. The expected reward for arm $a$ with context $x$ is $x^T \theta_a$. Ridge regression is solved independently per arm. Pros: simple, parallelizable. Cons: arms don't share information.

**Hybrid LinUCB**: Expected reward = $x^T \theta_a + z^T \beta$, where $z$ are shared features (e.g., user demographics) and $x$ are arm-specific features. Ridge regression is solved jointly. Pros: shares statistical strength across arms. Cons: more complex update rules.

**Yahoo! News Results** (Li et al., 2010, 50M+ impressions):

- LinUCB (disjoint): **12.5% CTR lift** over $\epsilon$-greedy.
- LinUCB (hybrid): marginal improvement over disjoint.
- Key insight: user features + article category features dramatically improve personalization.

### 9.2 Thompson Sampling with Context

**Linear-Gaussian Model**: Place a Gaussian prior $\theta \sim N(0, \Sigma_0)$. After $n$ observations of context-reward pairs $(x_i, r_i)$, the posterior is $N(\hat{\theta}_n, \Sigma_n)$ where:

$$\hat{\theta}_n = \Sigma_n \left(\Sigma_0^{-1} \theta_0 + \frac{1}{\sigma^2} X^T y\right), \quad \Sigma_n^{-1} = \Sigma_0^{-1} + \frac{1}{\sigma^2} X^T X$$

At each round, sample $\tilde{\theta} \sim N(\hat{\theta}_n, \Sigma_n)$ and pick $A_t = \arg\max_a f(x_t, a)^T \tilde{\theta}$.

**Logistic Model** (for binary rewards): No closed-form posterior. Use Laplace approximation (normal approximation at MAP) or MCMC sampling (e.g., Metropolis-Hastings).

### 9.3 Feature Engineering

| Feature Type | Examples |
|:-------------|:---------|
| **User features** | Age, gender, location, device, browser, browsing history, past CTR |
| **Item features** | Category, price, popularity, recency, text embeddings (BERT) |
| **Context features** | Time of day, day of week, season, traffic source |
| **Cross features** | User × item interactions, user category × time of day |
| **Derived features** | Click-through rate decay, popularity velocity, user affinity scores |

### 9.4 Offline Evaluation of Contextual Bandits

Evaluating bandit policies on logged data is challenging because the logging policy differs from the target policy.

- **Inverse Propensity Scoring (IPS)**: $\hat{V}_{IPS}(\pi) = \frac{1}{n} \sum_{i=1}^n r_i \frac{1\{A_i = \pi(x_i)\}}{\pi_0(A_i|x_i)}$. Unbiased if propensities are known.
- **Doubly Robust (DR) Estimation**: Combines a reward model $\hat{\mu}(x,a)$ with IPS correction: $\hat{V}_{DR} = \frac{1}{n} \sum_i \left[ \hat{\mu}(x_i, \pi(x_i)) + \frac{1\{A_i = \pi(x_i)\}}{\pi_0(A_i|x_i)} (r_i - \hat{\mu}(x_i, A_i)) \right]$. Lowers variance while maintaining (near) unbiasedness.
- **Rejection Sampling**: Only use log entries where $\pi_0(A_i|x_i) > 0$, reweight.

---

## 10. Implementation Considerations

### 10.1 $\epsilon$ Scheduling

- **Decay schedule**: $\epsilon_t = 1/t^p$ (e.g., $p=1/3$ for slower decay).
- **Adaptive $\epsilon$**: Increase $\epsilon$ when regret grows quickly; decrease when one arm dominance emerges.
- **Fixed $\epsilon$**: Simple, used in high-traffic settings where even $1\%$ exploration yields enough data.

### 10.2 Batch Updates

Update models in mini-batches (e.g., every 1000 events) instead of per-event. Reduces computational overhead and amortizes model retraining cost. Common in production systems with high event rates.

### 10.3 Exploration Budget

Separate traffic into **exploration** (experimental) and **exploitation** (production) channels. Common ratios: 90-95% exploit, 5-10% explore. Ensures business continuity if exploration policy performs poorly.

### 10.4 Cold Start

Before any data is collected, explore uniformly or use a prior:

- Use population-level priors (historical data from similar settings).
- Deploy Bayesian algorithms with uninformative priors.
- Multi-stage: pure exploration → $\epsilon$-greedy → Thompson Sampling.

### 10.5 Delayed Feedback

Rewards may arrive hours/days late (e.g., conversion after click). Strategies:

- **Optimistic imputation**: Assume unknown rewards are positive until proven otherwise.
- **Delayed UCB**: Only count observed rewards, correct for delay in confidence bounds.
- **Importance weighting**: Weight observed rewards by inverse probability of observation.

### 10.6 Non-Stationarity Detection

Monitor per-arm reward distributions for drift:

- **Statistical tests**: CUSUM, Page-Hinkley, Kolmogorov-Smirnov on rolling windows.
- **Reset strategy**: Reset arm statistics when change is detected, restart exploration.

### 10.7 Computational Efficiency

- **Sampling from posterior at scale**: Use Bootstrapped Thompson Sampling (sample from $B$ bootstrapped models) instead of exact MCMC.
- **Efficient UCB updates**: Maintain running sums $S_k = \sum r_t$, $N_k = \sum 1$ for $O(1)$ per-event updates.
- **Sparse updates**: For LinUCB with $d$-dimensional features, use Sherman-Morrison formula for $O(d^2)$ matrix inverse updates per event.

---

## 11. Tools and Libraries

| Library | Strengths | Algorithms | Language |
|:--------|:----------|:-----------|:---------|
| **Vowpal Wabbit** | Production-ready, billions of events/day, reduced to regression | LinUCB, Thompson Sampling, CB | C++ / Python |
| **TF-Agents (Google)** | Deep bandits, integration with TF ecosystem | NeuralBandit, LinUCB, Thompson Sampling | Python |
| **Azure Personalizer** | Managed cloud service, no infra needed | LinUCB-like (learns per user context) | REST API |
| **MABWiser** | Simulation, offline evaluation, interpretable | $\epsilon$-Greedy, UCB, Thompson, Exp3, LinUCB | Python |
| **Pyro (Uber)** | Bayesian inference at scale | Bayesian bandits with HMC/SVI | Python |
| **PyMC3 / PyMC** | MCMC and variational inference | Thompson sampling with custom models | Python |
| **Contextual* (R)** | Research-grade contextual bandit evaluation | LinUCB, Thompson, bootstrapped | R |
| **Spark Bandit** | Distributed bandits on Spark | $\epsilon$-Greedy, UCB (distributed) | Scala / PySpark |

### 11.1 Vowpal Wabbit Quick Example

Vowpal Wabbit reduces contextual bandits to cost-sensitive classification. A typical training line:

```
1:0.5:0.25 | user age:25 gender:1 | action category:news sports
```

Using `--cb_explore_adf` (explore then exploit) and `--cb_adf` (learn from logged data), VW handles exploration via Bagging, Softmax, or Epsilon exploration on top of any base learner.

### 11.2 Azure Personalizer

A fully managed contextual bandit service. Users define actions (features), context (features), and reward event. The service returns which action to show and learns from rewards. Uses a LinUCB-like algorithm with neural feature processing. Supports:
- Multi-slot (multiple actions per rank call)
- Delayed rewards (events can be reported up to 2 days later)
- Baseline actions (offline evaluation against logged policies)

---

## 12. Future Directions

### 12.1 Neural Contextual Bandits

Deep neural networks as reward predictors enable bandits to handle high-dimensional, non-linear features (images, text, user embeddings). Key approaches:

- **NeuralUCB / NeuralTS**: Neural network + last-layer uncertainty (via NTK or Laplace approximation).
- **Differentiable bandits**: End-to-end training of exploration via reparameterization tricks.

### 12.2 Bandit Meta-Learning

Learning to explore across related tasks. A meta-learner predicts which arms are likely promising based on past task experience. **Few-shot bandits**: after seeing a few tasks, rapidly adapt exploration strategy to new tasks.

### 12.3 Federated Bandits

Privacy-preserving bandits across decentralized data sources. Users' data never leaves their devices; only aggregated statistics (sufficient statistics for bandit updates) are shared. **Local differential privacy** mechanisms add noise to rewards before aggregation.

### 12.4 Bandits with LLMs

Large Language Models as powerful priors:

- **Cold-start acceleration**: LLM priors estimate reward expectations before any data is collected.
- **Action generation**: LLMs generate candidate actions (e.g., ad copy, recommendation explanations) in large or infinite action spaces.
- **Semantic exploration**: Use LLM embeddings to cluster actions and explore semantically similar groups.

### 12.5 Causal Bandits

Incorporate causal inference to answer counterfactual questions: "What would the reward have been if we had chosen arm $a$ instead?" **Instrumental variables**, **do-calculus**, and **causal graphs** guide exploration away from spurious correlations.

### 12.6 Multi-Agent Bandits

Multiple bandit agents interact in the same environment — competing for the same users, sharing information, or cooperating. Includes:

- **Competitive bandits**: Agents compete for rewards (e.g., two ad networks bidding for the same impression).
- **Cooperative bandits**: Agents share observations to accelerate learning.
- **Collaborative filtering as bandits**: Multiple users share arm preferences.

---

## 13. Code Examples

### 13.1 Epsilon-Greedy with Bernoulli Rewards

```python
import numpy as np

class EpsilonGreedy:
    def __init__(self, n_arms, epsilon=0.1):
        self.n_arms = n_arms
        self.epsilon = epsilon
        self.counts = np.zeros(n_arms, dtype=int)
        self.values = np.zeros(n_arms)  # empirical mean rewards

    def select_arm(self):
        if np.random.random() < self.epsilon:
            return np.random.randint(self.n_arms)
        return np.argmax(self.values)

    def update(self, arm, reward):
        self.counts[arm] += 1
        n = self.counts[arm]
        # Incremental update: new_avg = old_avg + (reward - old_avg) / n
        self.values[arm] += (reward - self.values[arm]) / n


def simulate(agent, n_rounds, arm_probs):
    rewards = np.zeros(n_rounds)
    for t in range(n_rounds):
        arm = agent.select_arm()
        reward = np.random.binomial(1, arm_probs[arm])
        agent.update(arm, reward)
        rewards[t] = reward
    return rewards

if __name__ == "__main__":
    arm_probs = [0.1, 0.3, 0.5, 0.7]
    e_greedy = EpsilonGreedy(n_arms=4, epsilon=0.1)
    rewards = simulate(e_greedy, 10_000, arm_probs)
    print(f"Epsilon-Greedy total reward: {rewards.sum():.0f}")
    print(f"Optimal would be: {0.7 * 10_000:.0f}")
    print(f"Regret: {0.7 * 10_000 - rewards.sum():.0f}")
    print(f"Arm pulls: {e_greedy.counts}")
```

### 13.2 UCB1 with Bernoulli Rewards

```python
import numpy as np

class UCB1:
    def __init__(self, n_arms):
        self.n_arms = n_arms
        self.counts = np.zeros(n_arms, dtype=int)
        self.values = np.zeros(n_arms)  # empirical means

    def select_arm(self, t):
        # Play each arm once first
        for arm in range(self.n_arms):
            if self.counts[arm] == 0:
                return arm
        # UCB selection
        bonus = np.sqrt(2 * np.log(t + 1) / self.counts)
        return np.argmax(self.values + bonus)

    def update(self, arm, reward):
        self.counts[arm] += 1
        n = self.counts[arm]
        self.values[arm] += (reward - self.values[arm]) / n


def simulate_ucb(agent, n_rounds, arm_probs):
    rewards = np.zeros(n_rounds)
    for t in range(n_rounds):
        arm = agent.select_arm(t)
        reward = np.random.binomial(1, arm_probs[arm])
        agent.update(arm, reward)
        rewards[t] = reward
    return rewards

if __name__ == "__main__":
    arm_probs = [0.1, 0.3, 0.5, 0.7]
    ucb = UCB1(n_arms=4)
    rewards = simulate_ucb(ucb, 10_000, arm_probs)
    print(f"UCB1 total reward: {rewards.sum():.0f}")
    print(f"Optimal would be: {0.7 * 10_000:.0f}")
    print(f"Regret: {0.7 * 10_000 - rewards.sum():.0f}")
    print(f"Arm pulls: {ucb.counts}")
```

### 13.3 Thompson Sampling with Bernoulli Rewards

```python
import numpy as np

class ThompsonSampling:
    """Bernoulli Thompson Sampling with Beta(1,1) prior."""
    def __init__(self, n_arms, alpha_prior=1, beta_prior=1):
        self.n_arms = n_arms
        self.alpha = np.full(n_arms, alpha_prior)   # successes + 1
        self.beta = np.full(n_arms, beta_prior)     # failures + 1

    def select_arm(self):
        samples = np.random.beta(self.alpha, self.beta)
        return np.argmax(samples)

    def update(self, arm, reward):
        self.alpha[arm] += reward       # success
        self.beta[arm] += 1 - reward    # failure


def simulate_ts(agent, n_rounds, arm_probs):
    rewards = np.zeros(n_rounds)
    for t in range(n_rounds):
        arm = agent.select_arm()
        reward = np.random.binomial(1, arm_probs[arm])
        agent.update(arm, reward)
        rewards[t] = reward
    return rewards

if __name__ == "__main__":
    arm_probs = [0.1, 0.3, 0.5, 0.7]
    ts = ThompsonSampling(n_arms=4)
    rewards = simulate_ts(ts, 10_000, arm_probs)
    print(f"Thompson Sampling total reward: {rewards.sum():.0f}")
    print(f"Optimal would be: {0.7 * 10_000:.0f}")
    print(f"Regret: {0.7 * 10_000 - rewards.sum():.0f}")
    print(f"Arm alpha (successes+1): {ts.alpha}")
    print(f"Arm beta  (failures+1): {ts.beta}")
```

### 13.4 Running Comparison

```python
def compare_algorithms(arm_probs, n_rounds=10_000, trials=100):
    results = {}
    for name, _ in [("EG(0.1)",0.1), ("EG(0.05)",0.05), ("UCB1",0), ("TS",0)]:
        regrets = []
        for _ in range(trials):
            if name.startswith("EG"): ag = EpsilonGreedy(4, float(name[3:-1]))
            elif name == "UCB1": ag = UCB1(4)
            else: ag = ThompsonSampling(4)
            rewards = simulate_ucb(ag, n_rounds, arm_probs) if name == "UCB1" else simulate(ag, n_rounds, arm_probs)
            regrets.append(max(arm_probs)*n_rounds - rewards.sum())
        results[name] = {"mean": np.mean(regrets), "std": np.std(regrets)}
    for name, r in sorted(results.items(), key=lambda x: x[1]["mean"]):
        print(f"{name:<15} Mean regret: {r['mean']:<10.1f} Std: {r['std']:.1f}")

compare_algorithms([0.1, 0.3, 0.5, 0.7])
```

**Sample output** (100 trials, 4 arms $p=[0.1,0.3,0.5,0.7]$): Thompson Sampling (~41) ≺ UCB1 (~68) ≺ EG(0.05) (~190) ≺ EG(0.1) (~388). TS consistently beats UCB1 and Epsilon-Greedy, with the gap widening as arms increase or means converge.

---

## References

1. **Auer et al. (2002)**. "Finite-time Analysis of the Multiarmed Bandit Problem." *Machine Learning*.
2. **Lai & Robbins (1985)**. "Asymptotically Efficient Adaptive Allocation Rules." *Advances in Applied Mathematics*.
3. **Li et al. (2010)**. "A Contextual-Bandit Approach to Personalized News Article Recommendation." *WWW 2010*.
4. **Agrawal & Goyal (2012)**. "Analysis of Thompson Sampling for the Multi-armed Bandit Problem." *COLT 2012*.
5. **Chapelle & Li (2011)**. "An Empirical Evaluation of Thompson Sampling." *NIPS 2011*.
6. **Russo et al. (2018)**. "A Tutorial on Thompson Sampling." *FnT Machine Learning*.
7. **Lattimore & Szepesvári (2020)**. *Bandit Algorithms*. Cambridge University Press.
8. **Slivkins (2019)**. "Introduction to Multi-Armed Bandits." *FnT Machine Learning*, 12(1-2).
9. **Burtini et al. (2015)**. "A Survey of Online Experiment Design with the Stochastic Multi-Armed Bandit." *arXiv:1510.00757*.
