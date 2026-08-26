# Mathematics for Machine Learning & Data Science — Study Notes

> **Topics:** Linear Algebra · Calculus · Probability · Statistics · Optimization · PCA

---

## 1. Linear Algebra

### 1.1 Vectors

**Definition:** $\mathbf{v} \in \mathbb{R}^n$ — ordered $n$-tuple of real numbers.

| Operation | Formula | Intuition |
|-----------|---------|-----------|
| Dot product | $\mathbf{a} \cdot \mathbf{b} = \sum a_i b_i = \|\mathbf{a}\|\|\mathbf{b}\|\cos\theta$ | Measures alignment; >0 = similar direction, =0 = orthogonal |
| L2 norm | $\|\mathbf{v}\|_2 = \sqrt{\sum v_i^2}$ | Euclidean length |
| L1 norm | $\|\mathbf{v}\|_1 = \sum_i \|v_i\|$ | Sparse/robust |
| Hadamard | $\mathbf{a} \odot \mathbf{b} = (a_1 b_1, \ldots)^T$ | Element-wise multiplication |
| Outer product | $\mathbf{a}\mathbf{b}^T$ | Rank-1 matrix |

**Linear Independence:** $\sum_i c_i \mathbf{v}_i = \mathbf{0} \implies c_i = 0 \ \forall i$. Max $n$ independent vectors in $\mathbb{R}^n$.

**Span / Basis:** All linear combinations of a set. A LI set that spans the space = basis. Standard basis: $\mathbf{e}_i = (0,\ldots,1,\ldots,0)^T$.

### 1.2 Matrices

**Definition:** $A \in \mathbb{R}^{m \times n}$ — $m$ rows, $n$ columns.

| Operation | Formula | Notes |
|-----------|---------|-------|
| Transpose | $(A^T)_{ij} = A_{ji}$ | $(AB)^T = B^T A^T$ |
| Matrix mult | $(AB)_{ij} = \sum_k A_{ik} B_{kj}$ | $A$: $m \times p$, $B$: $p \times n$ |
| Trace | $\operatorname{tr}(A) = \sum_i A_{ii} = \sum_i \lambda_i$ | Cyclic: $\operatorname{tr}(ABC)=\operatorname{tr}(BCA)$ |
| Determinant | $\det(A) = \prod_i \lambda_i$ | Volume scaling; $\det(AB)=\det(A)\det(B)$ |
| Inverse | $A^{-1}$ s.t. $AA^{-1}=I$ | Exists iff $\det(A)\neq 0$; $(AB)^{-1}=B^{-1}A^{-1}$ |

**Special Matrices:**

| Type | Property | Why It Matters |
|------|----------|----------------|
| Symmetric | $A = A^T$ | Real eigenvalues, orthogonal eigenvectors |
| Positive Definite | $\mathbf{x}^T A \mathbf{x} > 0 \ \forall \mathbf{x}\neq 0$ | All $\lambda_i > 0$; convexity; Cholesky |
| Orthogonal | $Q^T Q = I$ | Preserves length: $\|Q\mathbf{x}\| = \|\mathbf{x}\|$ |
| Diagonal | $D = \operatorname{diag}(d_1,\ldots,d_n)$ | Cheap invert & multiply |

**Rank:** Number of LI rows/columns. $\operatorname{rank}(A) \leq \min(m,n)$. Full rank = max possible.

**Nullspace:** $\{\mathbf{x} \mid A\mathbf{x} = \mathbf{0}\}$. Dimension = $n - \operatorname{rank}(A)$.

**Matrix Norms:** Frobenius $\|A\|_F = \sqrt{\sum A_{ij}^2} = \sqrt{\operatorname{tr}(A^T A)}$; Spectral $\|A\|_2 = \sigma_{\max}$.

### 1.3 Solving Linear Systems

$A\mathbf{x} = \mathbf{b}$: Square & invertible → $\mathbf{x}=A^{-1}\mathbf{b}$. Overdetermined ($m > n$) → least squares $\mathbf{x} = (A^T A)^{-1}A^T\mathbf{b}$. Underdetermined → min-norm $\mathbf{x} = A^T(AA^T)^{-1}\mathbf{b}$.

**Key Matrix Calculus Identities:**

| Derivative | Result | ML Use |
|-----------|--------|--------|
| $\nabla_\mathbf{x}(\mathbf{a}^T\mathbf{x})$ | $\mathbf{a}$ | Linear models |
| $\nabla_\mathbf{x}(\mathbf{x}^T A \mathbf{x})$ | $2A\mathbf{x}$ (symmetric $A$) | Quadratic forms |
| $\nabla_\mathbf{x}\|A\mathbf{x}-\mathbf{b}\|_2^2$ | $2A^T(A\mathbf{x}-\mathbf{b})$ | Least squares, linear regression |

### 1.4 Eigenvalues & Eigenvectors

$A\mathbf{v} = \lambda \mathbf{v}$. Characteristic: $\det(A - \lambda I) = 0$.

- $\operatorname{tr}(A) = \sum \lambda_i$, $\det(A) = \prod \lambda_i$
- Symmetric $A$ → all $\lambda_i \in \mathbb{R}$, eigenvectors orthogonal
- $A$ PD ⟺ all $\lambda_i > 0$; PSD ⟺ all $\lambda_i \geq 0$
- **Eigendecomposition:** $A = Q\Lambda Q^{-1}$. For symmetric: $A = Q \Lambda Q^T$ with $Q$ orthogonal.
- **Spectral radius:** $\rho(A) = \max_i \|\lambda_i\|$

**ML Relevance:** PCA = eigendecomposition of covariance matrix. Condition number of Hessian = $\lambda_{\max}/\lambda_{\min}$ governs GD convergence. Graph Laplacian eigenvectors → spectral clustering.

### 1.5 Singular Value Decomposition (SVD)

Any $A \in \mathbb{R}^{m \times n}$: $A = U \Sigma V^T$

- $U$ ($m \times m$): left singular vectors (orthogonal columns)
- $\Sigma$ ($m \times n$): $\sigma_1 \geq \sigma_2 \geq \cdots \geq \sigma_r > 0$
- $V$ ($n \times n$): right singular vectors (orthogonal columns)

**Connection to Eigendecomposition:**
- $AA^T = U \Sigma^2 U^T$ ($U$ = eigenvectors of $AA^T$)
- $A^T A = V \Sigma^2 V^T$ ($V$ = eigenvectors of $A^T A$)
- $\sigma_i = \sqrt{\lambda_i(A^T A)}$

**Key Properties:**
- $\operatorname{rank}(A) =$ count of non-zero singular values
- $\|A\|_F = \sqrt{\sum \sigma_i^2}$, $\|A\|_2 = \sigma_{\max}$
- Condition number $\kappa(A) = \sigma_{\max} / \sigma_{\min}$ — high $\kappa$ = ill-conditioned
- **Low-rank approx (Eckart-Young):** $A_k = U_k \Sigma_k V_k^T = \sum_{i=1}^k \sigma_i \mathbf{u}_i \mathbf{v}_i^T$ — best rank-$k$ approximation in Frobenius norm.

**ML Applications:** PCA, matrix completion, dimensionality reduction, denoising, pseudoinverse $A^+ = V \Sigma^+ U^T$.

---

## 2. Calculus

### 2.1 Derivatives

$f'(x) = \lim_{h\to 0} \frac{f(x+h)-f(x)}{h}$

| Rule | Formula |
|------|---------|
| Power | $\frac{d}{dx} x^n = n x^{n-1}$ |
| Product | $(fg)' = f'g + fg'$ |
| Quotient | $(f/g)' = (f'g - fg')/g^2$ |
| Chain | $(f\circ g)'(x) = f'(g(x))g'(x)$ |
| Exponential | $\frac{d}{dx} e^x = e^x$ |
| Log | $\frac{d}{dx} \ln x = 1/x$ |
| Sigmoid | $\sigma'(x) = \sigma(x)(1-\sigma(x))$ |
| Tanh | $\frac{d}{dx}\tanh x = 1 - \tanh^2 x$ |
| Softmax | $\partial \sigma_i / \partial x_j = \sigma_i(\delta_{ij} - \sigma_j)$ |

### 2.2 Partial Derivatives & Gradient

Partial: $\frac{\partial f}{\partial x_i}$ — derivative w.r.t. $x_i$ holding others constant.

**Gradient:** $\nabla f(\mathbf{x}) = (\partial f/\partial x_1, \ldots, \partial f/\partial x_n)^T$ — direction of **steepest ascent**; $-\nabla f$ = steepest descent. At min/max: $\nabla f = \mathbf{0}$. Gradient is orthogonal to level sets.

### 2.3 Jacobian & Hessian

**Jacobian** (vector → vector): $J_{ij} = \partial f_i/\partial x_j$ for $\mathbf{f}: \mathbb{R}^n \to \mathbb{R}^m$.

**Hessian** (scalar → matrix): $H_{ij} = \partial^2 f / \partial x_i \partial x_j$ (symmetric for smooth $f$).

| Hessian at critical point | Type |
|--------------------------|------|
| $H \succ 0$ (all $\lambda_i > 0$) | Local minimum |
| $H \prec 0$ (all $\lambda_i < 0$) | Local maximum |
| Indefinite (mixed signs) | Saddle point |

### 2.4 Chain Rule (Multivariate)

**Scalar composition:** $\frac{df}{dt} = \nabla f(g(t)) \cdot g'(t)$

**Vector composition ($z = f(\mathbf{y}), \mathbf{y}=g(\mathbf{x})$):** $\nabla_\mathbf{x} z = J_\mathbf{y}(\mathbf{x})^T \nabla_\mathbf{y} f$

This is **backpropagation** — chain rule applied layer by layer in neural networks.

### 2.5 Taylor Series

**Scalar:** $f(x) = f(a) + f'(a)(x-a) + \frac{1}{2}f''(a)(x-a)^2 + \cdots$

**Multivariate (2nd order):** $f(\mathbf{x}) \approx f(\mathbf{a}) + \nabla f(\mathbf{a})^T (\mathbf{x}-\mathbf{a}) + \frac{1}{2}(\mathbf{x}-\mathbf{a})^T H(\mathbf{a}) (\mathbf{x}-\mathbf{a})$

### 2.6 Common ML Derivatives

| Function | Derivative |
|----------|-----------|
| MSE: $\frac{1}{n}\sum(y_i - \hat{y}_i)^2$ | $\frac{2}{n}\sum(\hat{y}_i - y_i)$ |
| Cross-entropy: $-\sum y_i \log \hat{y}_i$ | $-\frac{y_i}{\hat{y}_i}$ |
| Log-likelihood (Gaussian) | $\frac{1}{\sigma^2}\sum(y_i - \mu)$ w.r.t. $\mu$ |
| L2 reg: $\lambda\|\mathbf{w}\|_2^2$ | $2\lambda\mathbf{w}$ |

---

## 3. Probability

### 3.1 Foundations

- **Sample space** $\Omega$: all possible outcomes. **Event:** subset of $\Omega$.
- **Axioms:** $P(A) \geq 0$, $P(\Omega) = 1$, $P(\cup A_i) = \sum P(A_i)$ for disjoint events.
- **Conditional:** $P(A \mid B) = P(A \cap B) / P(B)$
- **Independence:** $P(A \cap B) = P(A)P(B)$ ⟺ $A \perp B$
- **Law of Total Probability:** $P(A) = \sum_i P(A \mid B_i) P(B_i)$ for a partition $\{B_i\}$

### 3.2 Random Variables

$X: \Omega \to \mathbb{R}$.

- **PMF** (discrete): $p_X(x) = P(X=x)$
- **PDF** (continuous): $P(a \leq X \leq b) = \int_a^b f_X(x)\,dx$
- **CDF:** $F_X(x) = P(X \leq x)$ — non-decreasing, $\lim_{x\to -\infty}=0$, $\lim_{x\to \infty}=1$

### 3.3 Key Distributions

| Distribution | PMF/PDF | Mean | Var | Use |
|-------------|---------|------|-----|-----|
| **Bernoulli**$(p)$ | $p^x(1-p)^{1-x}$ | $p$ | $p(1-p)$ | Binary |
| **Binomial**$(n,p)$ | $\binom{n}{x}p^x(1-p)^{n-x}$ | $np$ | $np(1-p)$ | Count of successes |
| **Poisson**$(\lambda)$ | $\frac{\lambda^x e^{-\lambda}}{x!}$ | $\lambda$ | $\lambda$ | Rare events |
| **Uniform**$(a,b)$ | $1/(b-a)$ | $(a+b)/2$ | $(b-a)^2/12$ | Equal prob |
| **Gaussian**$(\mu,\sigma^2)$ | $\frac{1}{\sqrt{2\pi\sigma^2}} e^{-(x-\mu)^2/(2\sigma^2)}$ | $\mu$ | $\sigma^2$ | CLT, noise |
| **Exponential**$(\lambda)$ | $\lambda e^{-\lambda x}$ | $1/\lambda$ | $1/\lambda^2$ | Waiting times |
| **Beta**$(\alpha,\beta)$ | $x^{\alpha-1}(1-x)^{\beta-1}/B(\alpha,\beta)$ | $\frac{\alpha}{\alpha+\beta}$ | — | Conjugate for Bernoulli |

**Multivariate Gaussian:** $f(\mathbf{x}) = \frac{1}{(2\pi)^{d/2}|\Sigma|^{1/2}} \exp\left(-\frac12 (\mathbf{x}-\boldsymbol{\mu})^T \Sigma^{-1} (\mathbf{x}-\boldsymbol{\mu})\right)$

### 3.4 Moments

| Moment | Formula | Measures |
|--------|---------|----------|
| Mean $\mu$ | $\mathbb{E}[X] = \int x f(x)\,dx$ | Center |
| Variance $\sigma^2$ | $\mathbb{E}[(X-\mu)^2] = \mathbb{E}[X^2] - \mathbb{E}[X]^2$ | Spread |
| Skewness | $\mathbb{E}[(X-\mu)^3] / \sigma^3$ | Asymmetry |
| Kurtosis | $\mathbb{E}[(X-\mu)^4] / \sigma^4$ | Tail heaviness |

**Properties:** $\mathbb{E}[aX+b] = a\mathbb{E}[X]+b$; $\operatorname{Var}(aX+b) = a^2\operatorname{Var}(X)$; $\operatorname{Var}(X+Y) = \operatorname{Var}(X)+\operatorname{Var}(Y)+2\operatorname{Cov}(X,Y)$

### 3.5 Bayes' Theorem

$$P(\theta \mid \mathcal{D}) = \frac{P(\mathcal{D} \mid \theta) P(\theta)}{P(\mathcal{D})}$$

| Term | Name | Meaning |
|------|------|---------|
| $P(\theta \mid \mathcal{D})$ | **Posterior** | Belief after data |
| $P(\mathcal{D} \mid \theta)$ | **Likelihood** | Data probability given parameters |
| $P(\theta)$ | **Prior** | Belief before data |
| $P(\mathcal{D})$ | **Evidence** | Normalizing constant |

**Intuition:** Posterior ∝ Likelihood × Prior. This is how we update beliefs with evidence.

### 3.6 Maximum Likelihood Estimation (MLE)

$$\hat{\theta}_{\text{MLE}} = \arg\max_\theta \log P(\mathcal{D} \mid \theta)$$

| Distribution | MLE | Loss Function |
|-------------|-----|---------------|
| Bernoulli | $\hat{p} = \frac{1}{n}\sum x_i$ | Cross-entropy |
| Gaussian $\mu$ | $\hat{\mu} = \frac{1}{n}\sum x_i$ | MSE |
| Gaussian $\sigma^2$ | $\hat{\sigma}^2 = \frac{1}{n}\sum (x_i-\hat{\mu})^2$ | — |
| Linear regression | $\hat{\mathbf{w}} = (X^T X)^{-1} X^T \mathbf{y}$ | MSE (under Gaussian noise) |

**MAP (Maximum A Posteriori):** $\hat{\theta}_{\text{MAP}} = \arg\max [\log P(\mathcal{D}\mid\theta) + \log P(\theta)]$ — MLE + regularization (prior = regularizer). L2 = Gaussian prior on weights.

### 3.7 LLN & CLT

- **Law of Large Numbers:** $\bar{X}_n \xrightarrow{p} \mu$ — sample mean converges to expected value.
- **Central Limit Theorem:** $\frac{\bar{X}_n - \mu}{\sigma/\sqrt{n}} \xrightarrow{d} \mathcal{N}(0,1)$ — regardless of underlying distribution, the sampling distribution of the mean approaches Gaussian as $n \to \infty$.

---

## 4. Statistics

### 4.1 Descriptive Statistics

| Measure | Formula | Robust? |
|---------|---------|---------|
| Mean | $\bar{x} = \frac{1}{n}\sum x_i$ | No |
| Median | Middle value (sorted) | Yes |
| Mode | Most frequent | Yes |
| Variance | $s^2 = \frac{1}{n-1}\sum (x_i - \bar{x})^2$ (sample) | — |
| Std Dev | $s = \sqrt{s^2}$ | — |
| IQR | $Q_3 - Q_1$ | Yes |

**Five-number summary:** Min, Q1, Median, Q3, Max → box plot.

### 4.2 Correlation

**Pearson $r$:** $r = \frac{\operatorname{Cov}(X,Y)}{\sigma_X \sigma_Y} = \frac{\sum (x_i-\bar{x})(y_i-\bar{y})}{\sqrt{\sum (x_i-\bar{x})^2 \sum (y_i-\bar{y})^2}}$

- $r \in [-1, 1]$; $+1$ = perfect positive linear, $0$ = no linear, $-1$ = perfect negative
- Only measures **linear** relationships (Anscombe's quartet)
- **Spearman rank correlation:** Pearson on ranks — measures monotonic relationships

**Correlation ≠ Causation.**

### 4.3 Hypothesis Testing

**Framework:** $H_0$ (null, default) vs $H_1$ (alternative). Compute test statistic → p-value. Reject $H_0$ if p-value < $\alpha$ (typically 0.05).

| Error | Meaning | Prob |
|-------|---------|------|
| Type I | False positive (reject true $H_0$) | $\alpha$ |
| Type II | False negative (fail to reject false $H_0$) | $\beta$ |

**Power** $= 1 - \beta$ — probability of correctly rejecting false $H_0$.

| Test | Statistic | Use |
|------|-----------|-----|
| z-test | $z = (\bar{x} - \mu_0) / (\sigma/\sqrt{n})$ | Known variance, large $n$ |
| t-test (1-sample) | $t = (\bar{x} - \mu_0) / (s/\sqrt{n})$ | Unknown variance |
| t-test (2-sample) | $t = \frac{\bar{x}_1 - \bar{x}_2}{\sqrt{s_1^2/n_1 + s_2^2/n_2}}$ | Compare two groups |
| Paired t-test | $t = \bar{d} / (s_d/\sqrt{n})$ | Before/after same subjects |
| ANOVA (F-test) | $F = \frac{\text{between-group var}}{\text{within-group var}}$ | Compare $\geq 3$ group means |
| Chi-squared | $\chi^2 = \sum (O_i - E_i)^2 / E_i$ | Categorical association |

**95% CI for mean:** $\bar{x} \pm t_{\alpha/2, n-1} \cdot s/\sqrt{n}$

**p-value:** $P(\text{observed or more extreme data} \mid H_0)$. NOT $P(H_0 \mid \text{data})$.

### 4.4 Bias-Variance Tradeoff

$$\mathbb{E}[(y - \hat{f}(x))^2] = \underbrace{\text{Bias}[\hat{f}(x)]^2}_{\text{underfitting}} + \underbrace{\text{Var}[\hat{f}(x)]}_{\text{overfitting}} + \underbrace{\sigma^2}_{\text{irreducible}}$$

- High bias → underfitting (too simple). High variance → overfitting (too complex).
- Goal: minimize total expected error by trading off bias and variance.

---

## 5. Optimization

### 5.1 Gradient Descent

$$\mathbf{x}_{t+1} = \mathbf{x}_t - \eta \nabla f(\mathbf{x}_t)$$

**Intuition:** Step downhill in steepest direction. $\eta$ = learning rate.

**Convergence:** Convex $L$-smooth → $O(1/t)$. Strongly convex → linear $O(\rho^t)$. Oscillation when condition number $\kappa = L/\mu$ is large.

| Variant | Update | Why |
|---------|--------|-----|
| **SGD** | $\mathbf{w}_{t+1} = \mathbf{w}_t - \eta \nabla \mathcal{L}_i$ | One sample at a time; fast per-iteration |
| **Mini-batch** | $\mathbf{w}_{t+1} = \mathbf{w}_t - \frac{\eta}{B}\sum_{i\in B} \nabla \mathcal{L}_i$ | Batch of $B$; balances speed & stability |
| **Momentum** | $\mathbf{v}_{t+1} = \gamma \mathbf{v}_t + \eta\nabla f$, $\mathbf{x}_{t+1} = \mathbf{x}_t - \mathbf{v}_{t+1}$ | Accelerates, dampens oscillations |
| **Adam** | $m = \beta_1 m + (1-\beta_1)g$, $v = \beta_2 v + (1-\beta_2)g^2$, $\theta_{t+1} = \theta_t - \eta \hat{m}/(\sqrt{\hat{v}}+\epsilon)$ | Adaptive LR + momentum (default for DL) |

**LR scheduling:** Step decay, exponential decay, cosine annealing.

### 5.2 Convex Optimization

**Convex set:** $\lambda\mathbf{x} + (1-\lambda)\mathbf{y} \in C$ for any $\mathbf{x},\mathbf{y} \in C$, $\lambda \in [0,1]$.

**Convex function:** $f(\lambda\mathbf{x}+(1-\lambda)\mathbf{y}) \leq \lambda f(\mathbf{x}) + (1-\lambda)f(\mathbf{y})$

**Conditions:** 1st order: $f(\mathbf{y}) \geq f(\mathbf{x}) + \nabla f(\mathbf{x})^T(\mathbf{y}-\mathbf{x})$ (linear lower bound). 2nd order: $\nabla^2 f(\mathbf{x}) \succeq 0$ (PSD Hessian).

**Why Convexity Matters:** Every local minimum is global. No misleading saddle points. GD converges to global optimum. Strong duality for constrained problems.

**Convex examples:** Linear $a^T x + b$, quadratic $x^T Q x$ with $Q \succeq 0$, norms $\|\mathbf{x}\|_p$ ($p\geq 1$), negative log-likelihood of exponential families, max of convex functions.

**Non-convex examples:** Neural network loss surfaces, mixture models (unconstrained), $f(x,y)=x^2-y^2$.

### 5.3 Lagrange Multipliers & KKT Conditions

**Problem:** Minimize $f(\mathbf{x})$ s.t. $g_i(\mathbf{x}) = 0$, $h_j(\mathbf{x}) \leq 0$.

**Lagrangian:** $\mathcal{L}(\mathbf{x}, \boldsymbol{\lambda}, \boldsymbol{\mu}) = f(\mathbf{x}) + \sum \lambda_i g_i(\mathbf{x}) + \sum \mu_j h_j(\mathbf{x})$

**KKT Conditions:**

| Condition | Formula | Meaning |
|-----------|---------|---------|
| Stationarity | $\nabla f + \sum \lambda_i \nabla g_i + \sum \mu_j \nabla h_j = 0$ | Gradient of Lagrangian = 0 |
| Primal feasibility | $g_i(\mathbf{x}) = 0$, $h_j(\mathbf{x}) \leq 0$ | Constraints satisfied |
| Dual feasibility | $\mu_j \geq 0$ | Inequality multipliers non-negative |
| Slackness | $\mu_j h_j(\mathbf{x}) = 0$ | Either constraint active or multiplier zero |

**Duality:** Primal $p^* = \min_\mathbf{x} \max_{\boldsymbol{\mu}\geq 0,\boldsymbol{\lambda}} \mathcal{L}$, Dual $d^* = \max_{\boldsymbol{\mu}\geq 0,\boldsymbol{\lambda}} \min_\mathbf{x} \mathcal{L}$. Weak duality: $d^* \leq p^*$. Strong duality: $d^* = p^*$ (holds for convex problems with Slater's condition).

**ML Applications:** SVMs (max-margin via duality), regularization as constraint ($\min f(\mathbf{w})$ s.t. $\|\mathbf{w}\|_p \leq C$), entropy maximization.

### 5.4 Newton's Method

$$\mathbf{x}_{t+1} = \mathbf{x}_t - H(\mathbf{x}_t)^{-1} \nabla f(\mathbf{x}_t)$$

Second-order method using curvature. Quadratic convergence near min. Expensive: need $n\times n$ Hessian inverse. **Quasi-Newton** (BFGS, L-BFGS) approximate Hessian from gradient differences.

### 5.5 First-order Optimality

- **Necessary for local min:** $\nabla f(\mathbf{x}^*) = \mathbf{0}$, $\nabla^2 f(\mathbf{x}^*) \succeq 0$
- **Sufficient for strict local min:** $\nabla f(\mathbf{x}^*) = \mathbf{0}$, $\nabla^2 f(\mathbf{x}^*) \succ 0$

---

## 6. Principal Component Analysis (PCA)

### 6.1 The Goal

Given $X \in \mathbb{R}^{n \times d}$ ($n$ samples, $d$ features), find a low-dimensional representation preserving maximum variance.

**Two equivalent views:**
1. **Maximum variance:** Find $\mathbf{w}$ to maximize $\operatorname{Var}(X\mathbf{w})$ s.t. $\|\mathbf{w}\|=1$ → top eigenvector of $\operatorname{Cov}(X)$
2. **Min reconstruction error:** Minimize $\|X - X\mathbf{w}\mathbf{w}^T\|_F^2$ → same eigenvector

### 6.2 Algorithm

1. **Center data:** $\tilde{X} = X - \mathbf{1}\boldsymbol{\mu}^T$, $\boldsymbol{\mu} = \frac{1}{n}\sum \mathbf{x}_i$
2. **Covariance:** $\Sigma = \frac{1}{n-1} \tilde{X}^T \tilde{X}$
3. **Eigendecomposition:** $\Sigma = V \Lambda V^T$ where $V$ columns = PCs, $\Lambda = \operatorname{diag}(\lambda_1 \geq \cdots \geq \lambda_d)$
4. **Project:** $Z = \tilde{X} V_k$ ($n \times k$) — keep top $k$ PCs

### 6.3 Via SVD

PCA = SVD of centered data: $\tilde{X} = U \Sigma V^T$

- $V$ = principal directions (same as eigenvectors of $\Sigma$)
- $\sigma_i = \sqrt{(n-1)\lambda_i}$ (singular values → variance)
- Scores: $Z = \tilde{X} V = U \Sigma$
- SVD is numerically more stable than $X^T X$ eigendecomposition

### 6.4 Key Concepts

- **Variance explained by $k$th PC:** $\lambda_k / \sum_{j=1}^d \lambda_j$
- **Cumulative variance (top $k$):** $\sum_{j=1}^k \lambda_j / \sum_{j=1}^d \lambda_j$
- **Elbow rule:** Choose $k$ where cumulative variance plot bends
- **Standardize** features (to $z$-scores) before PCA if on different scales

### 6.5 Connections

| Method | Relationship |
|--------|-------------|
| PCR | PCA then regress on PCs |
| t-SNE / UMAP | Often apply PCA first for denoising |
| MDS | Classical MDS = PCA on distance matrix |
| Whitening | PCA + scale by $1/\sqrt{\lambda_i}$ → decorrelated unit-variance |

---

## 7. Quick-Reference Cheat Sheets

### 7.1 Linear Algebra

```
Dot:           a·b = ||a||||b||cos θ
Mat mult:      (AB)_ij = Σ_k A_ik B_kj
Inverse:       A^(-1) I = I A^(-1) = A
Trace:         tr(A) = Σ λ_i = Σ A_ii
Determinant:   det(A) = Π λ_i
Eigen:         A v = λ v,  A = Q Λ Q^(-1)
SVD:           A = U Σ V^T
Low-rank:      A_k = U_k Σ_k V_k^T
Pseudo-inv:    A⁺ = V Σ⁺ U^T
Condition #:   κ = σ_max / σ_min
```

### 7.2 Calculus

```
Chain:         d/dx f(g(x)) = f'(g(x)) g'(x)
Gradient:      ∇f = [∂f/∂x₁,…,∂f/∂xₙ]ᵀ  → steepest ascent
Jacobian:      J_ij = ∂f_i/∂x_j
Hessian:       H_ij = ∂²f/∂x_i ∂x_j
Taylor 2nd:    f(x) ≈ f(a) + ∇f·(x-a) + ½(x-a)ᵀH(x-a)
Sigmoid':      σ'(x) = σ(x)(1-σ(x))
Softmax':      ∂σ_i/∂x_j = σ_i(δ_ij - σ_j)
```

### 7.3 Probability

```
Bayes:         P(θ|D) = P(D|θ)P(θ) / P(D)
MLE:           θ̂ = argmax log P(D|θ)
MAP:           θ̂ = argmax [log P(D|θ) + log P(θ)]
LLN:           X̄_n → μ (in probability)
CLT:           (X̄ - μ)/(σ/√n) → N(0,1)
Var:           Var(X) = E[X²] - E[X]²
Cov:           Cov(X,Y) = E[(X-μ_X)(Y-μ_Y)]
Gaussian PDF:  N(x|μ,σ²) ∝ exp(-(x-μ)²/(2σ²))
```

### 7.4 Statistics

```
Pearson r:     r = Cov(X,Y)/(σ_X σ_Y)
Sample var:    s² = 1/(n-1) Σ(x_i - x̄)²
t-stat:        t = (x̄ - μ₀)/(s/√n)
CI 95%:        x̄ ± t_0.025·s/√n
p-value:       P(data | H₀) — NOT P(H₀ | data)
Type I err:    Reject true H₀ (α)
Type II err:   Fail to reject false H₀ (β)
Bias-var:      E[(y-ŷ)²] = Bias² + Var + σ²
```

### 7.5 Optimization

```
GD:            x_{t+1} = x_t - η ∇f(x_t)
Momentum:      v_{t+1} = γv_t + η∇f, x_{t+1} = x_t - v_{t+1}
Adam:          m = β₁m+(1-β₁)g, v = β₂v+(1-β₂)g²
               θ = θ - η·m̂/(√v̂+ε)
Newton:        x_{t+1} = x_t - H⁻¹∇f
Lagrangian:    L = f + Σλᵢgᵢ + Σμⱼhⱼ
KKT stat:      ∇f + Σμᵢ∇gᵢ + Σλⱼ∇hⱼ = 0
KKT slack:     μᵢgᵢ(x) = 0
Convex:        ∇²f(x) ⪰ 0
```

### 7.6 PCA

```
Center:        Ẋ = X - 𝟙μᵀ
Cov:           Σ = (1/(n-1)) ẊᵀẊ
Eigen:         Σ = VΛVᵀ  →  V = PCs
SVD:           Ẋ = UΣVᵀ  →  V same as above
Project:       Z = ẊV_k
Var expl:      λ_k / Σλⱼ
Choose k:      Elbow in scree plot
```

### 7.7 Notation Glossary

| Symbol | Meaning |
|--------|---------|
| $\mathbb{R}^n$ | $n$-dim real space |
| $\mathbf{x}$ | Column vector |
| $\|\mathbf{x}\|$ | Norm (L2 unless noted) |
| $A^T, A^{-1}, A^+$ | Transpose, inverse, pseudoinverse |
| $\operatorname{tr}(A), \det(A)$ | Trace, determinant |
| $A \succ 0, A \succeq 0$ | PD, PSD |
| $\nabla f, H$ | Gradient, Hessian |
| $\mathbb{E}[X], \operatorname{Var}(X)$ | Expectation, variance |
| $\operatorname{Cov}(X,Y)$ | Covariance |
| $\mathcal{N}(\mu, \sigma^2)$ | Gaussian distribution |
| $P(A \mid B)$ | Conditional probability |
| $\hat{\theta}$ | Estimated parameter |
| $\eta, \lambda$ | Learning rate, regularization coeff |
| $\sigma_i, \lambda_i$ | Singular values, eigenvalues |

### 7.8 Key Intuitions — One Sentence Each

| Concept | Intuition |
|---------|-----------|
| **Eigenvectors** | Directions that stay the same under a linear transformation |
| **SVD** | Factor any matrix into rotation × stretch × rotation |
| **Gradient** | Direction of steepest ascent; $-\nabla f$ = downhill |
| **Chain rule** | How to propagate derivatives through compositions (backprop) |
| **Bayes** | How to update beliefs with evidence |
| **MLE** | Parameters that make observed data most probable |
| **CLT** | Averages of any distribution look Gaussian for large $n$ |
| **p-value** | How surprising the data would be if $H_0$ were true |
| **Bias-Variance** | Simple models underfit (bias); complex models overfit (variance) |
| **Convexity** | Every local minimum is global; GD converges to the best solution |
| **KKT conditions** | At a constrained optimum, objective gradient balances constraint gradients |
| **PCA** | Find directions of maximum variance; project data onto them |

---

> **Suggested references:** *Mathematics for Machine Learning* (Deisenroth, Faisal, Ong), *The Elements of Statistical Learning* (Hastie et al.), *Pattern Recognition and Machine Learning* (Bishop), *Linear Algebra Done Right* (Axler), *Deep Learning* (Goodfellow et al.)
