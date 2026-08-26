# Customer Lifetime Value (CLV/LTV) Prediction — A Comprehensive Guide

> Traditional probabilistic models, ML approaches, feature engineering, churn integration, evaluation, and production patterns.

---

## 1. Foundations

**Customer Lifetime Value (CLV / LTV)** is the total net profit a business expects from a customer over the full relationship. The fundamental equation:

```
CLV = Σ (Revenue_t − Cost_t) / (1 + d)^t   for t = 0 to T
```

**Why CLV matters:**
- **Acquisition budgeting**: Max acceptable CPA = CLV. CLV:CAC ratio > 3:1 is standard SaaS health.
- **Tiered service**: High-CLV gets premium treatment; low-CLV gets automation.
- **Churn triage**: Combine CLV × churn probability to prioritize retention spend.
- **Product/pricing decisions**: CLV distributions reveal which segments generate long-term value.
- **Investor reporting**: CLV:CAC, payback period, net dollar retention.

**Contractual (subscription) vs. non-contractual (retail):**

| Aspect | Contractual | Non-Contractual |
|--------|------------|-----------------|
| Churn | Explicit cancel/non-renewal | Must be inferred from silence |
| Time | Billing cycles | Continuous, irregular transactions |
| Standard models | Survival / retention curves | BTYD (BG/NBD, Pareto/NBD) |
| Examples | SaaS, insurance, gyms | E-commerce, grocery, DTC |

---

## 2. Data Preparation

**Core schema:** `(customer_id, transaction_date, transaction_value)`.

**Derived per-customer aggregations** (at a calibration cut date):

| Metric | Description |
|--------|-------------|
| **Recency** | Days since last purchase |
| **Frequency** | Number of repeat purchases (total − 1) |
| **T** | Customer age — days since first purchase |
| **Monetary avg** | Mean transaction value |
| **Monetary std** | Variability in spend |

**Calibration/holdout split** — critical for all CLV evaluation:

```
┌────────────────────┬────────────────────┐
│  Calibration Period │   Holdout Period   │
│  (features + fit)   │   (evaluate CLV)   │
│  2023-01 ─── 2024-01 │ 2024-01 ─── 2024-07 │
└────────────────────┴────────────────────┘
```

**Rules:** Customers need ≥1 purchase in calibration. Holdout length matches the forecast horizon needed by the business. Remove test/employee accounts, handle returns and wholesale orders (cap at 3σ).

---

## 3. RFM Segmentation

**Recency, Frequency, Monetary** — three behavioral dimensions that correlate with future CLV.

### Scoring methods
- **Quintile-based**: Split each dimension into 1–5 bins. Code `534` means R=5, F=3, M=4.
- **Weighted RFM**: `CLV_score = 0.4R + 0.35F + 0.25M` (e-commerce typical).
- **K-means RFM**: Log-transform RFM, StandardScaler, K-means (k=4–8). Automated, data-driven.

### 11-segment action map

| Segment | R | F | M | Strategy |
|---------|---|---|---|----------|
| Champions | 4–5 | 4–5 | 4–5 | VIP, referrals, early access |
| Loyal | 2–5 | 3–5 | 3–5 | Upsell, cross-sell, rewards |
| Potential Loyalists | 3–5 | 1–3 | 1–3 | Nurture, education |
| New Customers | 4–5 | 0–1 | 0–1 | Onboarding, welcome |
| Needing Attention | 2–3 | 2–3 | 2–3 | Re-engagement |
| At Risk | 0–2 | 2–5 | 2–5 | Targeted retention |
| Can't Lose | 0–2 | 4–5 | 4–5 | Concierge retention |
| Lost | 0–1 | 0–1 | 0–1 | Suppress campaigns |

**Extensions:** RFMOC (+Onboarding, Category), RFMOCD (+Drop-off), LRFM (+Length).

---

## 4. Traditional Probabilistic Models (BTYD Family)

**Buy-Till-You-Die framework:** Each customer has a hidden purchase rate λ and a hidden "death" time. While alive, purchases follow a Poisson process with rate λ. At some point the customer permanently dies (churns). We infer P(Alive) from the pattern of observed transactions.

### 4.1 Pareto/NBD (Schmittlein, Morrison & Colombo, 1987)

- **Purchase rate λ** ~ Gamma(r, α) — Poisson while alive
- **Lifetime τ** ~ Exponential(μ), with μ ~ Gamma(s, β)
- "Pareto" = Pareto-distributed dropout; "NBD" = Negative Binomial marginal transaction count

**Pros:** Most theoretically sound BTYD model, gold-standard benchmark.
**Cons:** Likelihood involves Gaussian hypergeometric function — computationally expensive on large datasets.

### 4.2 BG/NBD (Fader, Hardie & Lee, 2005)

Dropout happens at transaction moments (Bernoulli p) rather than at a random continuous time.

- λ ~ Gamma(r, α) — purchase rate while alive
- p ~ Beta(a, b) — dropout probability after each transaction

**Pros:** Closed-form likelihood (beta functions), fast inference. Practically equivalent accuracy to Pareto/NBD.
**Cons:** Same strong distributional assumptions.

### 4.3 Gamma-Gamma (Monetary Value)

BTYD models predict **transaction count** only. Gamma-Gamma predicts **average transaction value**.

- Assumption: Monetary value is **independent** of the purchase process (must validate — often violated)
- z ~ Gamma(p, ν) per customer; ν ~ Gamma(q, γ)
- Output: `E[avg_transaction_value | observed_history]`

**Combined CLV:**
```
CLV = E[future_transactions] × E[avg_value] × margin
```

### 4.4 P(Alive) as a churn signal

Both models output P(Alive) — probability the customer is still active:
- **> 0.8**: Active — don't waste retention spend
- **0.3–0.8**: Uncertain — low-cost re-engagement
- **< 0.3**: Likely churned — aggressive win-back or suppress

### 4.5 Pareto/NBD with Covariates (pymc-marketing)

Allows customer features (channel, region) to influence λ and μ via log-link: `log(λ_i) = β_λ·X_i, log(μ_i) = β_μ·X_i`. Bayesian MCMC estimation.

---

## 5. Machine Learning Approaches

### 5.1 Why ML?

BTYD models make strong distributional assumptions. When violated, ML models capture non-linearity, interactions, and rich feature sets without parametric constraints.

### 5.2 Models in order of practical preference

| Model | Pros | Cons |
|-------|------|------|
| **XGBoost / LightGBM / CatBoost** | Non-linear, feature importance, handles missing data | Requires tuning |
| **Random Forest** | Robust, less tuning | Poor extrapolation |
| **Ridge / Lasso** | Interpretable, fast | Misses interactions |
| **ElasticNet** | Handles correlated features | Still linear |

### 5.3 Two-Stage (Hurdle) Model

CLV distributions are zero-inflated with long right tails. A single regression struggles.

```
Stage 1: P(purchase > 0 in holdout) → binary classifier (LogReg, XGBoost)
Stage 2: E[spend | purchase > 0] → regression on positive-only customers
CLV = P(purchase > 0) × E[spend | purchase > 0]
```

### 5.4 Quantile Regression

Predict specific percentiles instead of the mean — useful for CLV ranges:
- **10th percentile**: Conservative CLV for acquisition budgeting
- **50th percentile**: Expected CLV
- **90th percentile**: Upside potential

LightGBM/XGBoost support quantile objective natively.

### 5.5 Survival-Based ML

- **Random Survival Forest** — tree ensemble for censored data
- **CoxNet** — elastic-net Cox regression
- **DeepSurv** — neural hazard function

Apply to time-to-churn, then integrate survival curve × spend rate.

---

## 6. Deep Learning & Sequence Models

Use when you have **long transaction histories** (10+ transactions/customer) and **sequential patterns** matter.

### 6.1 LSTM / GRU

```
Input: [(t_1, v_1, f_1), ..., (t_n, v_n, f_n)] → LSTM → h_n → Dense → CLV
```

**Key design choices:** Encode time deltas (Δt) as features, log-transform values, pad short sequences with mask, concatenate running cumulative features at each timestep.

### 6.2 Seq2Seq (Jannach et al., 2021)

Encoder reads history → context vector → decoder generates future transaction sequence. Aggregate to CLV. Enables counterfactual simulation ("what if we accelerate the next purchase?").

### 6.3 Transformers

Self-attention captures long-range dependencies. Practical note: median retail transaction sequences are short (3–10) — transformers may be overkill.

### 6.4 Challenges with DL for CLV

| Challenge | Mitigation |
|-----------|-----------|
| Short histories (< 5 txns) | Use simpler models; DL needs 10+ to outperform GBMs |
| Irregular intervals | Time encoding + Phased LSTM |
| Heavy tails | Log-transform or Tweedie loss |
| Interpretability | SHAP / attention weights |
| Cold start | Separate fallback model |

---

## 7. Probabilistic vs. Deterministic Methods

### 7.1 The spectrum

```
Deterministic ─────────── Probabilistic
Simple avg   RFM heuristic   BG/NBD+GG   Bayesian BTYD
```

### 7.2 Deterministic

- **Simple historic**: `CLV = (Total revenue / Customer count) × Avg lifespan`. No per-customer differentiation.
- **RFM heuristic**: `CLV = w_R·R + w_F·F + w_M·M`. Transparent but no calibration to actual future value.

### 7.3 Probabilistic (BTYD, Bayesian)

Outputs distribution, not just point estimate. Advantages:
- **Uncertainty quantification**: Know when the model is unsure
- **Principled integration**: Churn and frequency modeled jointly
- **Data efficiency**: Parametric priors regularize sparse customers
- **Calibration**: Probabilistic predictions are well-calibrated if assumptions hold

Disadvantages: Strong distributional assumptions, hard to incorporate diverse feature types.

### 7.4 Hybrid approaches

| Strategy | How it works |
|----------|-------------|
| **BTYD + ML** | Use BG/NBD outputs (P(Alive), expected txns) as features in XGBoost |
| **Tweedie regression** | Single model for zero-inflated continuous target (XGBoost `reg:tweedie`) |
| **Bayesian DL** | Priors on NN weights via variational inference (Pyro, TF Probability) |

### 7.5 Model selection guide

| Scenario | Recommended |
|----------|------------|
| < 10k customers, interpretability critical | BG/NBD + Gamma-Gamma |
| 100k+ customers, many features | XGBoost (regression or two-stage) |
| Subscription business | Survival models |
| Need uncertainty intervals | Bayesian BTYD (PyMC-Marketing) |
| Sequential behavior, long histories | LSTM / Seq2Seq |
| Quick MVP | Weighted RFM |
| Regulated (explainability) | BTYD or linear GLM + SHAP |

---

## 8. Feature Engineering

### 8.1 Core RFM features

`recency_days`, `frequency`, `monetary_avg`, `monetary_total`, `customer_age_days`, `purchase_std`.

### 8.2 Temporal & behavioral

- **Interpurchase times**: mean, CV (regularity), max gap
- **Recency ratio**: recency / customer_age (normalized)
- **Weekend/morning purchases**: day-of-week / hour-of-day patterns
- **First purchase value & category**: anchor effect, intent signal
- **Periodicity score**: how regular the buying rhythm is
- **Gap to avg**: current silence relative to customer's own average

### 8.3 Rolling window aggregates

Compute RFM stats over multiple windows: 7d, 30d, 90d, 365d, all-time. Per window: sum, avg, max, min, std, count, recency, days active.

### 8.4 Ratio & cross features

- `avg_spend_30d / avg_spend_all` — spend acceleration
- `frequency_90d / frequency_all` — frequency trend
- `n_categories_purchased` — breadth of exploration
- `return_rate` — dissatisfaction signal
- `avg_discount_pct` — price sensitivity
- `promo_purchase_ratio` — reliance on promotions

### 8.5 Product, channel & interaction

- Category affinity (top-3), brand concentration (Herfindahl), SKU variety
- Acquisition channel, device, campaign source
- Support tickets (count, sentiment), email engagement (open/click rates)
- App sessions, feature adoption, referral count and conversion

### 8.6 BTYD-specific: just 3 numbers

BTYD models use only `frequency`, `recency`, and `T` — plus population parameters — to compute conditional expectations. This is both a strength (parsimony) and a limitation (cannot incorporate other features).

---

## 9. Churn Prediction Integration

### 9.1 Why they're inseparable

CLV = **value per period** × **expected remaining lifetime** (determined by churn). High spend + high churn ≤ moderate spend + loyal.

### 9.2 Integration patterns

**A — Sequential (BTYD):** BTYD → P(Alive) → conditional transactions + Gamma-Gamma → CLV.

**B — Two-stage ML:** Churn classifier → P(churn). CLV regression on non-churners. `CLV = (1 − P(churn)) × E[spend | no churn]`.

**C — Joint (implicit):** Single regression that predicts holdout spend directly. Zero-spend examples train the model to internalize churn.

**D — Survival:** Fit Cox / Random Survival Forest → predicted survival curve → expected remaining periods × spend per period.

### 9.3 Churn horizon alignment

| Use case | Churn horizon | Spend horizon |
|----------|--------------|--------------|
| Acquisition budgeting | 12–36 mo | Avg monthly spend × lifetime |
| Retention triage | 1–3 mo | Next-period spend |
| Quarterly reporting | 12 mo | Annual spend |
| Investor reporting | 24–60 mo | LTV/CAC |

Build one churn model per decision horizon. Feed appropriate probability into the corresponding CLV.

### 9.4 Churn-predictive features for CLV models

Recency, engagement decay (% change in frequency period-over-period), support interactions (recent complaints, unsubscribes), payment failures (subscription), reduced category diversity.

---

## 10. Model Evaluation & Validation

### 10.1 Calibration-holdout evaluation

**Target:** Actual total spend (or profit) in the holdout period.

| Metric | Formula | Notes |
|--------|---------|-------|
| MAE | mean(\|ŷ − y\|) | "Off by $X on average" |
| RMSE | sqrt(mean((ŷ−y)²)) | Penalizes large errors |
| MAPE | mean(\|ŷ−y\|/y)×100 | Breaks on zero values |
| R² | 1 − Σ(ŷ−y)²/Σ(ȳ−y)² | Standard fit |
| Spearman ρ | Rank correlation | Ordering quality |

### 10.2 Calibration assessment

**P(Alive):** Bin by predicted P(Alive) (0–0.1, 0.1–0.2…). In each bin, fraction who actually purchased in holdout should match predicted.

**Quantile calibration:** For predicted 10th percentile CLV, 10% of actual values should fall below.

### 10.3 Decile / lift analysis

Sort customers by predicted CLV descending → deciles → compute actual CLV per decile. **Top-decile lift** = actual CLV of top decile / average CLV. A good model shows monotonic decreasing lift. Top-decile lift is a common KPI.

### 10.4 Ranking metrics

When targeting quality matters more than absolute accuracy: Spearman ρ, Kendall's τ, NDCG, Precision@K.

### 10.5 Rolling-origin cross-validation

```
Fold 1: [Jan–Jun] → predict [Jul–Dec]
Fold 2: [Jan–Dec] → predict [Jan–Jun next year]
Fold 3: [Jan–Jun next year] → predict [Jul–Dec next year]
```

Guards against temporal drift.

### 10.6 Segment-level diagnostics

Evaluate per segment, not just globally. A model that looks good overall may systematically overpredict low-value customers and underpredict high-value ones.

### 10.7 Profit-based evaluation

Simulate the business impact: "If we use the CLV model to decide who gets a $10 retention offer (top 20% vs. random 20%), how much incremental profit does targeting generate?" ROI-based model comparison beats metric-only selection.

---

## 11. Implementation Patterns

### 11.1 Batch pipeline (common)

```yaml
Schedule: Daily/weekly
1. Extract transactions from warehouse
2. Recompute RFM + engineered features
3. Run model inference (BTYD: O(1) per customer; ML: batch predict on matrix)
4. Write results to CLV table: (customer_id, clv, p_alive, percentile, model_version)
5. CRM reads from CLV table for targeting
```

### 11.2 Near-real-time scoring

- **BTYD:** Update sufficient statistics (frequency, recency, monetary) incrementally per transaction. Recompute conditional expectation — cheap.
- **ML:** Store pre-computed feature vectors in Redis. After transaction, update rolling features via exponential decay, then predict.
- **DL:** Model server (TF Serving, TorchServe, ONNX). 10–50ms latency.

### 11.3 Retraining strategy

| Strategy | Frequency | When |
|----------|-----------|------|
| Batch retrain | Monthly | Stable behavior |
| Rolling retrain | Weekly | Seasonality, frequent promos |
| Triggered retrain | On drift | Monitor PSI/KL-divergence of predictions |

### 11.4 A/B testing CLV models

Control (current model) vs. treatment (new model). Metrics: Revenue per user, retention rate, marketing ROI over 2–3 forecast horizons. Guardrail: No decrease in satisfaction or short-term revenue.

### 11.5 Monitoring

- **Prediction drift (PSI)**: PSI > 0.1 → alert
- **Feature drift**: KS-test per feature vs. training → p < 0.01 → alert
- **Calibration drift**: Decile bias > 20% → alert
- **Pipeline failures**: > 1% failed predictions → alert

### 11.6 Production CLV table schema

```sql
CREATE TABLE customer_clv (
    customer_id        VARCHAR(64) PRIMARY KEY,
    predicted_clv_12m  DECIMAL(12,2),
    predicted_clv_24m  DECIMAL(12,2),
    p_alive            DECIMAL(5,4),
    expected_txns      DECIMAL(8,2),
    avg_txn_value      DECIMAL(10,2),
    clv_percentile     DECIMAL(5,2),
    prediction_date    DATE,
    model_version      VARCHAR(32),
    confidence_lower   DECIMAL(12,2),
    confidence_upper   DECIMAL(12,2)
);
```

---

## 12. Cold Start

**Zero-history:** Use demographic/account features (signup channel, plan tier, geo). Fall back to segment average by acquisition channel.

**Short history (< 3 txns):** Use shrinkage estimator: `CLV = w × individual + (1−w) × segment_avg`, with w growing as transaction count increases.

**Progressive personalization:**

```
Txns    Model state
   0    Segment average by channel
   1    + first-purchase value anchor
   2–3   Shrinkage estimator
   4+    Full individual BTYD or ML
  10+    Reliable individual estimates
```

---

## 13. Tools & Libraries

| Library | Focus | Key models |
|---------|-------|-----------|
| [lifetimes](https://github.com/CamDavidsonPilon/lifetimes) | BTYD (lightweight) | BG/NBD, Pareto/NBD, Gamma-Gamma |
| [PyMC-Marketing](https://www.pymc-marketing.io) | Bayesian CLV + MMM | BG/NBD, Pareto/NBD w/ covariates |
| XGBoost / LightGBM / CatBoost | Gradient boosting | Regression, survival, quantile, Tweedie |
| scikit-learn | General ML | Regression, RF, clustering |
| PyTorch / TensorFlow | Deep learning | LSTM, Transformer, Seq2Seq |
| lifelines | Survival analysis | Cox PH, Kaplan-Meier |
| CLVTools (R) | BTYD (comprehensive) | Pareto/NBD, BG/NBD, GG, extended |

### Pseudo-code: BG/NBD + Gamma-Gamma

```python
from lifetimes import BetaGeoFitter, GammaGammaFitter

summary = data.groupby('customer_id').agg(
    frequency=('date', lambda x: x.nunique()-1),
    recency=('date', lambda x: (x.max()-x.min()).days),
    T=('date', lambda x: (calib_end-x.min()).days),
    monetary_avg=('value','mean')
)

bgf = BetaGeoFitter().fit(summary['frequency'], summary['recency'], summary['T'])
summary['pred_txns'] = bgf.conditional_expected_number_of_purchases_up_to_time(365, ...)
summary['p_alive'] = bgf.conditional_probability_alive(...)

ggf = GammaGammaFitter().fit(summary['frequency'], summary['monetary_avg'])
summary['pred_value'] = ggf.conditional_expected_average_profit(...)

summary['clv_12m'] = summary['pred_txns'] * summary['pred_value']
```

### Pseudo-code: XGBoost two-stage

```python
import xgboost as xgb

X = compute_features(calib_data)
y = holdout_data.groupby('customer_id')['value'].sum().reindex(X.index, fill_value=0)

# Stage 1: purchase probability
clf = xgb.XGBClassifier(objective='binary:logistic').fit(X, y > 0)

# Stage 2: conditional spend
mask = y > 0
reg = xgb.XGBRegressor(objective='reg:tweedie').fit(X[mask], y[mask])

clv = clf.predict_proba(X)[:, 1] * reg.predict(X)
```

---

## 14. Key References

**Foundational papers:**
- Schmittlein, Morrison & Colombo (1987). "Counting Your Customers." *Management Science*. — Original Pareto/NBD.
- Fader, Hardie & Lee (2005). "Counting Your Customers the Easy Way." *Marketing Science*. — BG/NBD.
- Fader, Hardie & Shang (2010). "Customer-Base Analysis in a Discrete-Time Noncontractual Setting." *Marketing Science*. — BG/BB.

**ML & deep learning:**
- Jannach et al. (2021). "Improved CLV Prediction with Sequence-to-Sequence Learning." *ACM TKDD*.
- Chamberlain et al. (2022). "Scalable CLV Prediction using Gradient Boosted Trees." *Applied Marketing Analytics*.

**Resources:**
- [lifetimes documentation](https://lifetimes.readthedocs.io/)
- [PyMC-Marketing CLV Quickstart](https://www.pymc-marketing.io/en/stable/notebooks/clv/clv_quickstart.html)
- Fader, P. (2020). *Customer Centricity*. Wharton Digital Press.
- Hardie, B. (2014). *Customer-Base Analysis: A Practical Guide*.

---

> **Maintenance note**: This guide was written in 2026. For the latest developments — deep learning advances, Bayesian BTYD in PyMC, and production MLOps patterns — consult the referenced libraries and arXiv (search: `customer lifetime value`, `buy till you die`, `CLV prediction`).
