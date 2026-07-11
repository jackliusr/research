# Data Warehouse Modelling for Gaming Platforms: Players, Bets, and Game Recommendations

> **Domain**: Gaming & Betting Data Engineering  
> **Scope**: Kimball dimensional modelling, Data Vault, ML features for game recommendation  
> **Audience**: Data engineers, architects, ML engineers

---

## 1. Domain Overview

### 1.1 Key Entities

| Entity | Description | Characteristics |
|--------|-------------|-----------------|
| **Players** | User profiles with demographics, VIP tier, KYC/AML status | Millions, SCD, PII |
| **Games** | Catalog — slots, table, sportsbook, live casino | Thousands of SKUs, metadata (RTP, volatility) |
| **Bets** | Wagering transactions — singles, accumulators, systems | Billions, high-velocity, complex lifecycle |
| **Outcomes** | Results (win/loss/void/cashout) | Delayed arrival (sports settle hours/days later) |
| **Recommendations** | ML-driven game suggestions | Feature vectors, model predictions, A/B tests |

### 1.2 Key Business Metrics

| Metric | Definition | Question |
|--------|------------|----------|
| **Handle** | Sum of all bet amounts | How much activity? |
| **GGR** | Total bets − total payouts | House revenue? |
| **NGR** | GGR − bonuses − taxes − fees | Actual profit? |
| **Player LTV** | Forecast net revenue over lifetime | Which players to invest in? |
| **Win/Loss Ratio** | Player winnings ÷ total wagered | Are games balanced? |
| **Retention Rate** | % active in N returning in N+1 | Platform stickiness? |

### 1.3 Data Sources

```
Game Engine (real-time)    → Kafka/Flink ──┐
CRM (player profiles)      ───────────────┐ ├── Staging (Parquet/S3)
Payment System (financial) ───────────────┼┤
Odds Feed (dynamic pricing) ─────────────┘└── Raw Layer
External (fixtures, meta) ─────────────────┘
```

- **Game engine**: Real-time Kafka events. 10K–100K+ events/sec during peak.
- **CRM**: Player registration, profile updates, KYC, marketing attribution.
- **Payment**: Deposits, withdrawals, chargebacks, currency conversion. Audit-critical.
- **Odds feed**: Dynamic pricing — changes second-by-second. High cardinality.
- **External**: Sports fixtures, venues, game metadata providers.

### 1.4 Scale Considerations

- **High-velocity**: 50,000+ bets/sec during major events (Super Bowl). Streaming mandatory.
- **Late-arriving data**: Bet placed today settles 72h later. Placement date ≠ settlement date.
- **SCD game attributes**: RTP/volatility updated post-release. Historical analysis needs values *at play time*.
- **Multi-currency**: Deposit in SGD, bet in EUR, report in USD. Capture exchange rate at bet time.
- **Regulatory**: 5–10 year retention of transaction audit trails.
- **Volume**: Mid-tier platform produces 1–5 TB/day raw data.

---

## 2. Dimensional Model (Kimball Star Schema)

### 2.1 Fact Tables

#### Fact_Bet — Core Transaction Fact

**Grain**: One row per bet. **Partitioning**: Daily on `bet_date_sk`.

| Column | Type | Notes | Additivity |
|--------|------|-------|------------|
| `bet_sk` | BIGINT PK | Surrogate | — |
| `bet_id` | VARCHAR(64) | Unique natural key | — |
| `player_sk`, `game_sk` | FK | Who, what game | — |
| `bet_type_sk` | FK | Single, accumulator | — |
| `bet_date_sk`, `settlement_date_sk` | FK → Dim_Date | Placement & settlement | — |
| `currency_sk`, `market_sk`, `promotion_sk`, `device_channel_sk` | FK | Currency, market, promo, device | — |
| `bet_amount`, `bet_amount_base` | DEC(18,2) | Stake (orig + base) | **Additive** |
| `actual_payout`, `win_loss_amount` | DEC(18,2) | Payout & net | **Additive** |
| `odds_at_bet_time`, `exchange_rate` | DEC | Snapshot values at bet time | **Semi-additive** |
| `bet_status` | VARCHAR(20) | open/won/lost/voided | — |
| `recommendation_source` | VARCHAR(32) | Model that served this bet | — |

```sql
CREATE TABLE fact_bet (
    bet_sk BIGINT IDENTITY(1,1) PRIMARY KEY,
    bet_id VARCHAR(64) NOT NULL UNIQUE,
    player_sk INT NOT NULL, game_sk INT NOT NULL,
    bet_type_sk INT NOT NULL, bet_date_sk INT NOT NULL,
    settlement_date_sk INT, currency_sk INT NOT NULL,
    market_sk INT, promotion_sk INT, device_channel_sk INT,
    bet_amount DECIMAL(18,2) NOT NULL,
    bet_amount_base DECIMAL(18,2),
    odds_at_bet_time DECIMAL(10,4),
    actual_payout DECIMAL(18,2),
    win_loss_amount DECIMAL(18,2),
    exchange_rate DECIMAL(12,6),
    bet_status VARCHAR(20) DEFAULT 'open',
    days_to_settlement INT,
    recommendation_source VARCHAR(32),
    etl_batch_id VARCHAR(32)
) PARTITION BY RANGE (bet_date_sk);
```

#### Fact_Bet_Event — Accumulating Snapshot

**Grain**: One row per bet tracking lifecycle (placement → settlement → payout).  
**Use**: Settlement SLA monitoring, payout ops.

| Column | Description |
|--------|-------------|
| `bet_sk` | BIGINT PK — same surrogate as Fact_Bet |
| `placed_date_sk`, `settled_date_sk`, `voided_date_sk`, `payout_date_sk` | FK→Dim_Date — lifecycle dates |
| `current_status`, `current_odds`, `days_to_settlement`, `num_status_changes` | Current state & metrics |

#### Fact_Player_Daily — Periodic Snapshot

**Grain**: One row per player per day. **Use**: Engagement, cohort retention, churn.

```sql
CREATE TABLE fact_player_daily (
    player_sk INT NOT NULL, date_sk INT NOT NULL,
    total_bets INT DEFAULT 0, total_bet_amount DECIMAL(18,2) DEFAULT 0,
    total_payout DECIMAL(18,2) DEFAULT 0, net_win_loss DECIMAL(18,2) DEFAULT 0,
    session_count INT DEFAULT 0, total_session_duration INT DEFAULT 0,
    distinct_games_played INT DEFAULT 0,
    deposit_amount DECIMAL(18,2) DEFAULT 0,
    withdrawal_amount DECIMAL(18,2) DEFAULT 0,
    has_played BOOLEAN DEFAULT FALSE,
    PRIMARY KEY (player_sk, date_sk)
);
```

#### Fact_Player_Lifetime — Accumulating Snapshot

**Grain**: One row per player. UPSERT on activity. **Use**: VIP tier assignment, LTV.

```sql
CREATE TABLE fact_player_lifetime (
    player_sk INT PRIMARY KEY,
    first_bet_date DATE, last_bet_date DATE,
    first_deposit_date DATE,
    lifetime_bets BIGINT DEFAULT 0,
    lifetime_bet_amount DECIMAL(18,2) DEFAULT 0,
    lifetime_payout DECIMAL(18,2) DEFAULT 0,
    lifetime_net_revenue DECIMAL(18,2) DEFAULT 0,
    lifetime_deposits DECIMAL(18,2) DEFAULT 0,
    lifetime_withdrawals DECIMAL(18,2) DEFAULT 0,
    lifetime_games_played INT DEFAULT 0,
    days_since_last_bet INT,
    estimated_ltv DECIMAL(18,2)
);
```

### 2.2 Dimension Tables

#### Dim_Player

| Column | SCD | Notes |
|--------|-----|-------|
| `player_sk` | — | Surrogate PK |
| `player_id` | — | Natural key |
| `email`, `phone` | Type 1 | Corrections only |
| `registration_date`, `country`, `date_of_birth`, `marketing_channel` | Type 0 | Immutable |
| `vip_tier`, `account_status`, `kyc_status`, `risk_flag` | **Type 2** | Track history for attribution |
| `age_group`, `preferred_currency`, `device_type` | Type 1/2 | |
| `effective_date`, `expiry_date`, `is_current` | — | SCD metadata |

**Type 2 rationale**: A VIP player in Q1 downgraded in Q2 needs different historical attribution. Type 1 for email/phone (operational). Type 0 for country/DOB.

#### Dim_Game

| Column | SCD | Notes |
|--------|-----|-------|
| `game_sk` | — | Surrogate PK |
| `game_id` | — | Natural key |
| `game_name` | Type 1 | Display name |
| `game_category`, `provider`, `theme` | Type 2 | slots, table, live_casino |
| `rtp`, `volatility` | **Type 2** | Can change post-release |
| `is_active`, `mobile_optimized` | Type 2 | |
| `min_bet`, `max_bet`, `jackpot_flag` | Type 1 | Operational |
| `release_date` | Type 0 | |

**Why Type 2 on RTP/volatility**: If RTP changes from 96.2% to 96.5%, historical win/loss must use the value at play time. Bridge table for multi-category games: `bridge_game_category(game_sk, category_code, weight)`.

#### Dim_Bet_Type

| Code | Name | Category | Settlement |
|------|------|----------|------------|
| `SINGLE` | Single Bet | straight | deferred |
| `ACC4` | 4-Fold Accumulator | parlay | deferred |
| `LUCKY15` | Lucky 15 (15 bets) | system | deferred |
| `LIVE_SINGLE` | Live In-Play Single | live | instant |

#### Dim_Market (Sportsbook)

Columns: `market_sk`, `market_code`, `sport`, `league`, `event_name`, `event_date`, `period`, `is_live`.

**Cardinality warning**: Can reach millions. Consider fact-dimensional hybrid — static attrs (sport, league) in dim, event attrs as degenerate fact columns.

#### Dim_Date

```sql
CREATE TABLE dim_date (
    date_sk INT PRIMARY KEY, full_date DATE NOT NULL,
    day_of_week INT, day_name VARCHAR(16),
    month_number INT, month_name VARCHAR(16),
    quarter INT, year INT,
    is_weekend BOOLEAN, is_holiday BOOLEAN DEFAULT FALSE,
    season VARCHAR(20), fiscal_year INT, fiscal_quarter INT
);
```

Populate 20+ years. Seasonality (World Cup, Super Bowl) drives massive volume.

#### Dim_Promotion

SCD Type 2. Columns: `promotion_sk`, `promotion_id`, `type` (welcome_bonus, reload, free_bet, cashback), `value`, `wagering_requirements`, `start_date`, `end_date`.

#### Dim_Device_Channel

Columns: `device_channel_sk`, `device_type` (mobile/desktop/tablet), `operating_system`, `browser`, `channel` (web/app/in_play).

---

## 3. Data Vault Alternative

### 3.1 When Data Vault Suits Gaming

Data Vault (Linstedt) excels with: (1) multiple heterogeneous sources (game engine + CRM + payments + odds feed), (2) frequent schema changes, (3) full audit trail for financial transactions.

### 3.2 Hubs (Business Keys)

| Hub | Business Key |
|-----|-------------|
| `Hub_Player` | `player_id` |
| `Hub_Game` | `game_id` |
| `Hub_Bet` | `bet_transaction_id` |
| `Hub_Promotion` | `promotion_id` |

```sql
CREATE TABLE hub_player (
    player_hk CHAR(32) PRIMARY KEY,    -- MD5 of business key
    player_id VARCHAR(64) NOT NULL UNIQUE,
    load_date TIMESTAMP NOT NULL,
    record_source VARCHAR(64) NOT NULL
);
```

### 3.3 Links (Relationships)

| Link | Connected Hubs |
|------|---------------|
| `Link_Bet_Player_Game` | Hub_Bet + Hub_Player + Hub_Game |
| `Link_Player_Promotion` | Hub_Player + Hub_Promotion |
| `Link_Game_Market` | Hub_Game + Hub_Market |

### 3.4 Satellites (Context)

| Satellite | Parent | Attributes | SCD |
|-----------|--------|------------|-----|
| `Sat_Player_Profile` | Hub_Player | Demographics, email, country | Type 1 |
| `Sat_Player_VIP` | Hub_Player | VIP tier, account status, KYC, risk | Type 2 |
| `Sat_Game_Attributes` | Hub_Game | RTP, volatility, theme, provider | Type 2 |
| `Sat_Bet_Details` | Link_Bet_Player_Game | Amount, odds, payout, status | Type 1 |

```sql
CREATE TABLE sat_player_vip (
    player_hk CHAR(32) NOT NULL REFERENCES hub_player(player_hk),
    load_date TIMESTAMP NOT NULL, record_source VARCHAR(64) NOT NULL,
    vip_tier VARCHAR(32), account_status VARCHAR(20),
    kyc_status VARCHAR(20), risk_flag VARCHAR(20),
    effective_date DATE NOT NULL, expiry_date DATE DEFAULT '9999-12-31',
    is_current BOOLEAN DEFAULT TRUE,
    hash_diff CHAR(32) NOT NULL,
    PRIMARY KEY (player_hk, load_date)
);
```

### 3.5 Data Vault → Kimball Bridge

```
Data Vault → Point-in-Time (PIT) tables → Star Schema (for BI)
```

PIT tables resolve SCD Type 2 lookups to a single snapshot per day.

---

## 4. Training Data for Game Recommendation

### 4.1 Feature Store Design

Every bet in `Fact_Bet` is **implicit feedback**: the player chose game X over alternatives.

```sql
CREATE TABLE ml_features_player_game (
    player_sk INT REFERENCES dim_player(player_sk),
    game_sk INT REFERENCES dim_game(game_sk),

    -- Player features
    player_total_bets INT, player_avg_bet_amount DECIMAL(18,2),
    player_win_rate DECIMAL(5,4), player_favorite_category VARCHAR(64),
    player_avg_session_duration INT, player_days_since_last_bet INT,
    player_days_since_registration INT, player_total_deposits DECIMAL(18,2),
    player_active_days_30d INT, player_bet_frequency_7d DECIMAL(10,2),
    player_vip_tier VARCHAR(32), player_age_group VARCHAR(20), player_country CHAR(2),

    -- Game features
    game_popularity_rank INT, game_avg_bet_count_7d INT,
    game_unique_players_7d INT, game_win_rate_30d DECIMAL(5,4),
    game_category VARCHAR(64), game_volatility VARCHAR(16),
    game_rtp DECIMAL(5,2), game_provider VARCHAR(64),
    game_theme VARCHAR(64), game_jackpot_flag BOOLEAN,
    game_days_since_release INT,

    -- Interaction features
    player_game_bet_count INT, player_game_total_won DECIMAL(18,2),
    player_game_last_played DATE, player_game_days_since_last INT,
    player_game_avg_bet DECIMAL(18,2), player_game_bet_count_7d INT,
    player_game_win_rate DECIMAL(5,4),

    -- Temporal & promotional
    hour_of_day INT, day_of_week INT, is_weekend BOOLEAN,
    player_has_active_promotion BOOLEAN, player_on_streak INT,

    -- Target
    will_play_next_7d BOOLEAN,
    PRIMARY KEY (player_sk, game_sk)
);
```

### 4.2 Recommendation Approaches

#### Collaborative Filtering (Spark ALS)

Input: player × game implicit feedback matrix.

| Rating Option | Formula | Pros | Cons |
|--------------|---------|------|------|
| Bet count | `COUNT(bets)` | Simple | Ignores stake |
| Total wagered | `SUM(bet_amount)` | Economic preference | Skewed |
| Derived | `LOG(1+SUM(bet_amount))` | Smoothed | Complex |

```python
from pyspark.ml.recommendation import ALS
als = ALS(userCol="player_idx", itemCol="game_idx",
          ratingCol="bet_count_log", implicitPrefs=True,
          rank=50, regParam=0.1, alpha=40.0, coldStartStrategy="drop")
model = als.fit(training_data)
```

**Rating matrix SQL**:
```sql
SELECT p.player_sk, g.game_sk,
       COUNT(fb.bet_sk) AS bet_count,
       LOG(1+COUNT(fb.bet_sk)) AS bet_count_log,
       SUM(fb.bet_amount) AS total_wagered
FROM fact_bet fb
JOIN dim_player p ON fb.player_sk=p.player_sk AND p.is_current=TRUE
JOIN dim_game g ON fb.game_sk=g.game_sk AND g.is_current=TRUE
WHERE fb.bet_date_sk >= 20240101
GROUP BY p.player_sk, g.game_sk;
```

#### Content-Based Filtering

Player profile = weighted average of game feature vectors. New games ranked by cosine similarity.

```sql
SELECT fb.player_sk,
       AVG(g.rtp) AS pref_rtp,
       MODE() WITHIN GROUP (ORDER BY g.game_category) AS pref_category,
       MODE() WITHIN GROUP (ORDER BY g.provider) AS pref_provider
FROM fact_bet fb JOIN dim_game g ON fb.game_sk=g.game_sk
GROUP BY fb.player_sk;
```

#### Hybrid (LightFM)

```python
from lightfm import LightFM, Dataset
dataset = Dataset()
dataset.fit(users=player_ids, items=game_ids,
    user_features=['vip_tier','country','age_group'],
    item_features=['category','provider','volatility'])
(interactions, _) = dataset.build_interactions(
    [(r.player_id, r.game_id, r.bet_count) for r in bet_data])
model = LightFM(learning_rate=0.05, loss='warp', no_components=64)
model.fit(interactions, epochs=30)
```

TensorFlow Recommenders: two-tower retrieval (user + item tower → candidates) + deep ranking.

#### Sequence-Aware (GRU4Rec / SASRec)

```sql
SELECT player_sk, game_sk, bet_timestamp,
       ROW_NUMBER() OVER (PARTITION BY player_sk ORDER BY bet_timestamp) AS seq_pos,
       LAG(game_sk,1) OVER (PARTITION BY player_sk ORDER BY bet_timestamp) AS prev_game,
       LEAD(game_sk,1) OVER (PARTITION BY player_sk ORDER BY bet_timestamp) AS next_game
FROM fact_bet WHERE bet_date_sk >= @train_start;
```

GRU4Rec for session-level prediction; SASRec for longer user history with self-attention.

### 4.3 Training Data Pipeline

```
Source           Feature Pipeline                 ML Pipeline
──────           ─────────────────                ───────────
Fact_Bet  ──►  Window Aggregations             Feature Store
Dim_Player──►    → Player Features          ──►  (Feast/Tecton)
Dim_Game  ──►    → Game Features                 + Online Store
Dim_Date  ──►    → Player-Game Interaction       (Redis/DynamoDB)
                 + Temporal + Promotional              │
                       │                          Model Training
                       ▼
              Time-based split:
              Train: M1-M9, Val: M10, Test: M11
```

**Negative sampling**: 4–10 negatives per positive (stratified by game popularity).

### 4.4 Feature Engineering Summary

| Group | Examples | Source | Refresh |
|-------|----------|--------|---------|
| Demographics | Age, country, VIP tier | Dim_Player | Daily |
| Engagement | Total bets, days active, avg bet size | Fact_Player_Daily | Daily |
| Recency | Days since last bet/session | Fact_Player_Daily | Daily |
| Financial | Deposits, withdrawals, net win/loss | Fact_Player_Daily | Daily |
| Popularity | Bet count (7d/30d), unique players, win rate | Fact_Bet (agg) | Daily |
| Game attributes | Category, volatility, RTP, provider | Dim_Game | Weekly |
| Player-game history | Bet count, win, last played | Fact_Bet (per pair) | Daily |
| Temporal | Hour, DOW, weekend, season | Dim_Date | Static |
| Promotional | Bets with promo, bonus usage | Fact_Bet + Dim_Promotion | Daily |
| Rolling aggs | 7d/14d/30d moving averages | Fact_Bet (window) | Daily |

### 4.5 Data Leakage Prevention

| Anti-Pattern | Why Leakage | Fix |
|-------------|-------------|-----|
| `player_total_bets` includes future bets | Feature uses data after prediction time | Cutoff date: features as of D, target = D+1:D+8 |
| `game_bet_count_7d` includes current player | Own future bets shouldn't inform recs | Exclude current player from agg |
| Random train/test split | Future bets leak into training | Always split by time |
| Lifetime `last_bet_date` | Includes post-row-date bets | Use daily snapshot or rolling windows |

**Point-in-time correct**:
```sql
SELECT fb.player_sk, fb.game_sk, fb.bet_date_sk AS cutoff,
       fpd.total_bets AS player_total_sofar,
       game_pop.g7d_bet_count AS game_pop_7d,
       CASE WHEN COUNT(next.bet_sk)>0 THEN TRUE ELSE FALSE END AS will_play_next_7d
FROM fact_bet fb
LEFT JOIN fact_player_daily fpd
    ON fb.player_sk=fpd.player_sk AND fpd.date_sk=fb.bet_date_sk-1
LEFT JOIN (SELECT game_sk,bet_date_sk,COUNT(*) g7d_bet_count
           FROM fact_bet GROUP BY game_sk,bet_date_sk) game_pop
    ON fb.game_sk=game_pop.game_sk AND game_pop.bet_date_sk=fb.bet_date_sk-1
LEFT JOIN fact_bet next
    ON fb.player_sk=next.player_sk AND fb.game_sk=next.game_sk
    AND next.bet_date_sk BETWEEN fb.bet_date_sk+1 AND fb.bet_date_sk+7
GROUP BY fb.player_sk,fb.game_sk,fb.bet_date_sk,fpd.total_bets,game_pop.g7d_bet_count;
```

### 4.6 Cold Start Strategies

| Scenario | Strategy |
|----------|----------|
| **New player, existing games** | Content-based fallback: geo-popular games + preferences |
| **Existing player, new game** | Exploration bonus: `score = predicted + exp(-days_since_release/14)` |
| **New player, new game** | Global popularity + demographics similarity |
| **New category** | Multi-armed bandit: 5-10% exploration slots |

---

## 5. ETL/ELT Considerations

### 5.1 Late-Arriving Facts

Bets settle hours/days later. Pattern:
```
Day 1: INSERT Fact_Bet (bet_status='open', settlement_date_sk=NULL)
Day 3: UPDATE settlement_date_sk, actual_payout, bet_status='won' WHERE bet_id='...'
```

One row per bet — UPDATE maintains grain integrity. For audit trail, use Data Vault append-only satellites.

### 5.2 High-Volume Ingestion

```
Game Engine → Kafka → Flink Streaming
  ├── Real-time aggs (Redis)
  ├── Hourly micro-batch fact loads
  └── Daily batch dimension SCD loads
```

### 5.3 Odds Changes

Store `odds_at_bet_time` in Fact_Bet. Keep `current_odds` in Redis. Do NOT build a rapid-change dim — cardinality too high for OLAP benefit.

### 5.4 Multi-Currency

```
Fact_Bet: currency_sk='EUR', bet_amount=100, exchange_rate=1.18, bet_amount_base=118.00 (USD)
```

Daily exchange rate snapshots in `Dim_Exchange_Rate`.

### 5.5 Player Attribution

- **First-touch** (Dim_Player.marketing_channel, immutable): acquisition channel.
- **Last-touch** (Fact_Bet.device_channel_sk): bet-time channel.

Both answer different questions. Store both.

---

## 6. Reporting & Analytics Use Cases

| Report | Fact Tables | Dimensions | Measures |
|--------|------------|------------|----------|
| **Daily GGR** | Fact_Bet | Dim_Date, Dim_Game | SUM(bet_amount−payout) |
| **Cohort Retention** | Fact_Player_Daily | Dim_Date (cohort) | COUNT(DISTINCT player), % |
| **Game Popularity** | Fact_Bet | Dim_Game, Dim_Date | COUNT(*), SUM(bet_amount) |
| **VIP Performance** | Fact_Bet + Fact_Player_Lifetime | Dim_Player (VIP tier) | SUM(net_revenue), COUNT(bets) |
| **Promotion ROI** | Fact_Bet (promotion_sk) | Dim_Promotion, Dim_Date | SUM(bet_amount), SUM(payout) |
| **Recommendation A/B** | Fact_Bet (source) | Dim_Game, source | CTR, conversion, avg_bet |
| **Settlement SLA** | Fact_Bet_Event | Dim_Bet_Type, Dim_Game | AVG(days_to_settlement), P95 |

### Daily GGR

```sql
SELECT d.full_date, g.game_name, g.game_category,
       COUNT(*) bet_count, SUM(fb.bet_amount_base) handle,
       SUM(fb.win_loss_amount) ggr
FROM fact_bet fb JOIN dim_date d ON fb.bet_date_sk=d.date_sk
JOIN dim_game g ON fb.game_sk=g.game_sk AND g.is_current=TRUE
WHERE fb.bet_status IN ('won','lost')
GROUP BY d.full_date, g.game_name, g.game_category;
```

### Cohort Retention (Jan 2026)

```sql
WITH cohort AS (
    SELECT player_sk FROM dim_player
    WHERE registration_date BETWEEN '2026-01-01' AND '2026-01-31'
)
SELECT DATEDIFF('week', dp.registration_date, fpd.date_sk) week,
       COUNT(DISTINCT fpd.player_sk) active_players
FROM cohort c JOIN dim_player dp ON c.player_sk=dp.player_sk
JOIN fact_player_daily fpd ON c.player_sk=fpd.player_sk
WHERE fpd.has_played=TRUE GROUP BY week ORDER BY week;
```

---

## 7. Anti-Patterns & Gotchas

### 7.1 Counting Same Bet Multiple Times

Accumulators stored as multiple rows → inflated counts. **Fix**: One row per bet. Separate `Fact_Bet_Leg` for leg-level detail.

### 7.2 Mixing Bet and Session Data

Session measures in Fact_Bet. **Fix**: Keep Fact_Bet (transaction) and Fact_Player_Daily (session) separate.

### 7.3 Ignoring Settlement Timing

GGR reported using only placement date. **Fix**: Placement date for marketing, settlement date for revenue. Use `WHERE settlement_date_sk = @today` for financials.

### 7.4 Over-Normalizing Game Attributes

Separate Dim_Provider, Dim_Theme, Dim_Category. **Fix**: Games change rarely. Denormalize into one Dim_Game.

### 7.5 Underestimating Fact Cardinality

No partitioning → TB scans. **Fix**: Partition by date. Sort by `(bet_date_sk, player_sk)`. Add aggregate facts.

### 7.6 Cold Start for Recommendations

CF returns nothing for new items. **Fix**: Content-based fallback + exploration bonus + bandit.

### 7.7 Trusting Source Data Quality

Loading raw events without validation. **Fix**: Filter test bets, validate `bet_amount > 0`, flag anomalies (1000 identical bets in 1 min), use `bet_id` UNIQUE for dedup.

---

## 8. References

**Books**: Kimball & Ross — *The Data Warehouse Toolkit, 3rd Ed.* (gaming chapter). Linstedt & Olschimke — *Data Vault 2.0.*

**Docs**: [Spark ALS](https://spark.apache.org/docs/latest/mllib-collaborative-filtering.html), [TF Recommenders](https://www.tensorflow.org/recommenders), [LightFM](https://github.com/lyst/lightfm), [Feast](https://feast.dev)  
**Papers**: GRU4Rec (Hidasi et al., ICLR 2016), SASRec (Kang & McAuley, ICDM 2018)

**Industry**: SBTech/DraftKings blog, Kambi Group talks, Evolution Gaming infrastructure, Tecton blog

---

> **Version**: 1.0 (July 2026)
