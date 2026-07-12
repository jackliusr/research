# Gambling Datasets: A Comprehensive Guide for Machine Learning and Analytics

> A curated reference on publicly available gambling and betting datasets, covering sports betting, casino games, poker, esports, responsible gambling research, synthetic data generation, data quality challenges, and legal/ethical considerations.

**Author:** Research Agent — Jack Liu Shurui
**Date:** July 2026
**Scope:** Open, publicly accessible datasets suitable for ML modelling, analytics research, and academic study. Commercial/proprietary data sources are noted where relevant.

---

## Table of Contents

1. Overview
2. Sports Betting Datasets
3. Casino Games Datasets
4. Poker Datasets
5. Esports Datasets
6. Responsible Gambling Research Datasets
7. Synthetic Data Generation for Gambling
8. Data Quality Challenges
9. Legal & Ethical Considerations
10. Sources & Further Reading

---

## 1. Overview

Gambling generates vast quantities of structured behavioural data: bet amounts, game outcomes, player session logs, odds movements, and financial transactions. For ML practitioners and researchers, these datasets offer rich ground for:

- **Predictive modelling** (match outcomes, player value, churn)
- **Anomaly detection** (match-fixing, money laundering, problem gambling markers)
- **Strategy optimisation** (bet sizing via Kelly Criterion, portfolio theory)
- **Behavioural analytics** (responsible gambling interventions, player segmentation)
- **Game AI** (reinforcement learning agents for poker, blackjack, esports)

This guide catalogues publicly available datasets across every major gambling vertical, details synthetic data approaches for privacy-preserving research, and documents the data quality and ethical challenges unique to this domain.

---

## 2. Sports Betting Datasets

Sports betting is the most dataset-rich gambling vertical because match-level statistics have been collected systematically for decades outside of gambling contexts.

### 2.1 Multi-Sport Betting & Odds

| Dataset / Source | Coverage | Key Features | Access |
|---|---|---|---|
| **Kaggle Sports Betting Datasets** | NFL, NBA, MLB, NHL, soccer | Betting lines, game results, player stats | [kaggle.com](https://kaggle.com/datasets?search=sports+betting) |
| **football-data.co.uk** | 20+ soccer leagues (1990s–present) | Match results, 1X2 odds, over/under, Asian handicaps | Free CSV download |
| **The Odds API** | NFL, NBA, MLB, NHL, soccer, tennis | Live + historical odds from 70+ bookmakers; free tier | [the-odds-api.com](https://the-odds-api.com/) |
| **Betfair Historical Data** | Exchange markets (soccer, horse racing, tennis) | Tick-by-tick backtesting data | [historicdata.betfair.com](https://historicdata.betfair.com/) |
| **Sports Reference** | MLB, NFL, NBA, NHL, NCAA | Box scores, play-by-play, advanced metrics | [sports-reference.com](https://www.sports-reference.com/) |
| **FiveThirtyEight Data** | NFL, NBA, MLB, soccer | ELO ratings, game predictions, historical forecasts | [GitHub](https://github.com/fivethirtyeight/data) |

### 2.2 League-Specific Datasets

**NFL:**
- **nflverse/nfldata** — Weekly-updated play-by-play, roster, schedule data since 1999. GitHub: nflverse/nfldata.
- **nflverse/nflfastR-data** — Pre-cleaned EPA, WPA, CPOE metrics. GitHub: nflverse/nflfastR-data.
- **Kaggle NFL Big Data Bowl** — Annual NFL Next Gen Stats tracking data (player coordinates, speed, acceleration).
- **NFL Scores & Betting Data** (Kaggle) — Spreads and over/unders from 1966 to present.

**NBA:**
- **Basketball Reference** — Complete historical stats with advanced metrics (PER, WS, BPM). Scrapable via `nba_api`.
- **NBA Betting Dataset** (Kaggle) — Historical stats merged with betting lines and totals.
- **NBA Games Dataset** (Kaggle, nathanlauga) — Game outcomes, team stats, rankings.
- **NBA Shot Charts** — Shot-level coordinates from `stats.nba.com`.

**MLB:**
- **Lahman Baseball Database** — Player/team stats from 1871 to present. The gold standard open baseball dataset.
- **Retrosheet** — Play-by-play for every MLB game since 1921.
- **Statcast (Baseball Savant)** — Pitch-level tracking: exit velocity, launch angle, spin rate, expected stats (xBA, xSLG).

**Soccer:**
- **European Soccer Database** (Kaggle, hugomathien) — 25,000+ matches from 11 European leagues with player attributes from FIFA games.
- **FBref** — Advanced soccer stats from StatsBomb (expected goals, pressing stats).
- **Open Football Data** — Free open public domain football data covering leagues and cups worldwide.
- **Transfermarkt** — Player valuations, transfer history, squad data.

**Other Sports:**
- **Tennis Abstract** — Match-by-match results with ELO ratings and serve stats.
- **UFC Stats** — Official striking, grappling, and fight outcomes.
- **Hockey Reference** — NHL game logs, advanced metrics (Corsi, Fenwick, xG).

### 2.3 R & Python Libraries for Data Retrieval

- **Python:** `sportsipy`, `nba_api`, `nfl_data_py`, `pybaseball`, `sportsreference`, `cfbd`, `hockey_scraper`
- **R:** `nflfastR`, `hoopR`, `wehoop`, `baseballr`, `worldfootballR`, `hockeyR`

### 2.4 Python Toolkit Example

```python
# Retrieve NBA betting data using sportsipy
from sportsipy.nba.teams import Teams
from sportsipy.nba.schedule import Schedule

# Historical schedule with results
schedule = Schedule('GSW')
for game in schedule:
    print(game.date, game.opponent, game.points_scored, game.points_allowed)

# Fetch odds from The Odds API (requires API key)
import requests
resp = requests.get(
    'https://api.the-odds-api.com/v4/sports/basketball_nba/odds/',
    params={'apiKey': 'YOUR_KEY', 'regions': 'us', 'markets': 'spreads'}
)
```

---

## 3. Casino Games Datasets

Public casino datasets are rarer than sports datasets because real casino transaction data is proprietary and commercially sensitive. Existing datasets are predominantly simulated.

### 3.1 Blackjack

| Dataset | Size | Description | Link |
|---|---|---|---|
| **50 Million Blackjack Hands** | 50M simulated hands | Realistically simulated blackjack rounds with dealer up-card, player sum, actions, outcomes | [Kaggle (dennisho)](https://kaggle.com/datasets/dennisho/blackjack-hands) |
| **Blackjack RL Dataset** | 887K hands | Used for Monte Carlo RL agent training; web app with real-time learning | [GitHub (chirayu08)](https://github.com/chirayu08/blackjack-rl) |

The 50 Million Blackjack Hands dataset is the most comprehensive public casino dataset. Fields include: player hand value, dealer up-card, whether the player hit/stayed, and final outcome (win/loss/push). Useful for training basic strategy agents and evaluating card-counting simulations.

### 3.2 Real Transaction Data

The **Payments Transaction Data from Online Casino Players and Online Sports Bettors** dataset (Data in Brief, 2023) contains one year of anonymised customer payment records from:

- A casino-focused gambling brand
- A sports-focused gambling brand

Fields: deposit amounts, withdrawal amounts, transaction timestamps, payment method types. The raw data was sourced from a US-based digital gambling payments systems provider. Available through the ScienceDirect article [S2352340923001956](https://www.sciencedirect.com/science/article/pii/S2352340923001956).

### 3.3 Simulated Casino Data

For ML practitioners who need custom casino datasets, game-level simulation is the standard approach:

- **Roulette:** Full probability distribution is known — no need for historical data. Simulate spins with `numpy.random.choice` over 37/38 outcomes.
- **Slot machines:** Parameterised RTP (return-to-player) percentages combined with random payout distributions.
- **Baccarat:** Simple combinatorial probability — shoe composition driving hand outcomes.
- **Craps:** Dice-outcome probability tables are exact.

The `sports-betting` Python package ([georgedouzas.github.io/sports-betting](https://georgedouzas.github.io/sports-betting/)) provides dataloaders and bettor objects for creating and testing betting models, including casino-style wagers.

---

## 4. Poker Datasets

Poker is uniquely dataset-rich because of both academic interest (imperfect-information game AI) and the widespread practice of hand history logging.

### 4.1 UCI Poker Hand Dataset

- **Records:** 1,025,010 hands
- **Classes:** 10 poker hand rankings (nothing, pair, two pair, three of a kind, straight, flush, full house, four of a kind, straight flush, royal flush)
- **Features:** 10 categorical attributes (suit + rank for each of 5 cards)
- **Challenge:** Extreme class imbalance — 50.1% nothing, 42.3% one pair, remaining 8 classes account for only 7.6%
- **Uses:** Benchmarking classification algorithms on categorical-only, highly imbalanced data; educational ML exercises

Available at: [archive.ics.uci.edu/dataset/158/poker+hand](https://archive.ics.uci.edu/dataset/158/poker+hand)

### 4.2 Poker Hand History (PHH) Dataset

The University of Toronto Computer Poker Research Group's PHH dataset is the largest public poker corpus:

- **Total hands:** ~620 million (full Zenodo archive)
- **Sources:**
  - 341M heads-up/3-player fixed-limit + 279M heads-up no-limit Texas hold'em hands from the **Annual Computer Poker Competition** (2009–2017)
  - 21.6M uncorrupted no-limit hold'em hands from anonymised real-world hand history logs (July 2009) comprising hands from PokerStars (3.1M), PartyPoker (8.3M), iPoker Network (6.0M), and others
  - All 83 televised hands from the 2023 WSOP $50K Poker Players Championship final table
  - All 10,000 hands played by Pluribus (Brown & Sandholm, 2019), the first AI to defeat elite human pros in multi-player no-limit hold'em
- **Format:** PHH (Poker Hand History) standardised format — arXiv 2312.11753
- **Access:** GitHub (uoftcprg/phh-dataset); full archive at [Zenodo doi:10.5281/zenodo.10796885](https://doi.org/10.5281/zenodo.10796885)

### 4.3 Key Poker Datasets Summary

| Dataset | Hands | Format | Best For |
|---|---|---|---|
| UCI Poker Hand | 1.0M | 10 categorical features + label | Classification benchmarks |
| PHH (GitHub) | 21.6M (real) + 620M (simulated) | PHH format | RL, game theory, strategy mining |
| ACPC Competition | 620M | Variants, utility estimates | AI agent evaluation |
| Pluribus Hands | 10K | No-limit hold'em | Multi-player AI analysis |
| WSOP 2023 Final Table | 83 hands | Mixed games | Qualitative strategy study |

### 4.4 PHH Format Example

```text
Game: NLH
Seat 1: PlayerA (100bb)
Seat 2: PlayerB (100bb)
PlayerA posts SB 0.5bb
PlayerB posts BB 1bb
Preflop:
PlayerA raises to 3bb
PlayerB calls 2bb
Flop: [Kd][7c][2h]
PlayerB checks
PlayerA bets 4bb
PlayerB folds
```

---

## 5. Esports Datasets

Esports betting data is less standardised than traditional sports but growing rapidly with the professionalisation of competitive gaming.

### 5.1 Available Datasets

| Dataset / Source | Games Covered | Key Features | Access |
|---|---|---|---|
| **Kaggle Esports Datasets** | Dota 2, LoL, CS:GO, Overwatch | Match results, player stats, map scores | [kaggle.com](https://kaggle.com/datasets?search=esports) |
| **PandaScore API** | LoL, Dota 2, CS:GO, Valorant, Overwatch | Live odds, match schedules, player stats, historical results | [pandascore.co](https://pandascore.co/) |
| **game-datasets (GitHub)** | 100+ games | Curated list of game AI/datamining datasets | [leomaurodesenv/game-datasets](https://github.com/leomaurodesenv/game-datasets) |
| **OpenDota API** | Dota 2 | Full match data, player histories, hero stats | [opendota.com](https://www.opendota.com/) |
| **Riot Games API** | League of Legends | Match timelines, champion stats, summoner data | [developer.riotgames.com](https://developer.riotgames.com/) |
| **HLTV.org** | CS:GO / Counter-Strike | Match results, player ratings, map stats | Scrapable |
| **Liquipedia** | Multiple esports | Tournament results, team histories, prize pools | [liquipedia.net](https://liquipedia.net/) |

### 5.2 Key Considerations for Esports Betting Models

- **Patch cycles:** Game balance patches (every 2–4 weeks for LoL, Dota 2) shift meta and invalidate historical data
- **Roster changes:** Team performance is highly sensitive to player substitutions
- **Map/code variance:** CS:GO map pools rotate; LoL champion pick/ban phases create combinatorial complexity
- **Data quality:** Unofficial APIs may have rate limits, incomplete coverage, or undocumented breaking changes

---

## 6. Responsible Gambling Research Datasets

These datasets support academic research into problem gambling detection, harm minimisation, and player protection.

### 6.1 The Transparency Project

The **Transparency Project** (Division on Addiction, Cambridge Health Alliance / Harvard Medical School) is the most significant public repository of real gambling behaviour data.

- **Data source:** Collaborative research project between the Division on Addiction and bwin (now Entain plc)
- **Available datasets:**
  - *Population Trends in Internet Sports Gambling* — Longitudinal sports betting data with deposit/withdrawal records, daily betting aggregates, and demographic information (age, gender, country, language)
  - *Internet Poker Gambling* — Real poker behaviour data supporting studies on gambling trajectories (LaPlante et al., 2009)
- **Access:** Password-protected at [thetransparencyproject.org](http://thetransparencyproject.org); researchers must request credentials
- **Key variables:** Monetary deposits, withdrawals, betting activity aggregates, session durations, game types

### 6.2 NGAGE 2.0

The **National Survey of Gambling Attitudes and Gambling Experiences** (National Council on Problem Gambling, 2021) provides detailed US population-level data:

- National prevalence rates for gambling participation and problem gambling
- Attitudes toward sports betting expansion
- Demographic breakdowns by age, gender, income, region
- Available at: ncpgambling.org

### 6.3 Other Research Sources

| Source | Focus | Access |
|---|---|---|
| **Harmony Discovery** | UK-based gambling behaviour studies aggregator | [harmonydata.ac.uk](https://harmonydata.ac.uk) |
| **GREO Evidence Centre** | Problem gambling prediction models, machine learning studies | [greo.ca](https://www.greo.ca) |
| **Bournemouth University** | Responsible gambling interaction modalities | Research publications |
| **Journal of Gambling Studies** | Transactional behaviour analysis, risk markers | Springer |

### 6.4 Predicting Problem Gambling with ML

Recent studies (Sandor & Bako, 2024; Greo) use ML on transactional data to identify at-risk gamblers:

- **Features:** Deposit frequency, bet size variability, session duration, chasing behaviour, time of day, game-type switching
- **Target:** Problem Gambling Severity Index (PGSI) scores, self-exclusion events, account-level harm markers
- **Modelling approaches:** Random forest, gradient boosting, LSTM for session sequences, anomaly detection on deviation from individual baselines

---

## 7. Synthetic Data Generation for Gambling

Real gambling data is heavily restricted by privacy regulations and commercial sensitivity. Synthetic data generation is both a solution and a research domain in its own right.

### 7.1 Methods

| Method | Description | Best For | Tools |
|---|---|---|---|
| **CTGAN** (Conditional GAN) | GAN-based tabular data synthesis; handles mixed data types | Transaction records, player demographics | sdv-dev/CTGAN (GitHub) |
| **TVAE** (Variational Autoencoder) | VAE-based single-table synthetic data | Continuous behavioural features | sdv-dev/CTGAN |
| **SDG-GAN** | GAN framework for fraud detection datasets | Gambling fraud and money laundering patterns | arXiv 2109.12546 |
| **Agent-Based Simulation** | Rule-based synthetic player generation | Casino game outcomes, betting flows | Mesa (Python), AnyLogic |
| **Statistical Bootstrapping** | Resampling with privacy-preserving noise | Small research datasets | numpy, scipy |

### 7.2 CTGAN for Gambling Data

The Synthetic Data Vault (SDV) ecosystem provides the most accessible CTGAN implementation:

```python
from sdv.single_table import CTGANSynthesizer
from sdv.datasets.local import load_csvs

# Load real gambling transaction data
real_data = load_csvs(folder_path='./gambling_data/')

# Train synthesizer
synthesizer = CTGANSynthesizer(metadata)
synthesizer.fit(real_data)

# Generate 10,000 synthetic records
synthetic_data = synthesizer.sample(10000)
```

### 7.3 Applications

- **Privacy-preserving research:** Synthesise datasets that preserve aggregate statistical properties (bet distributions, churn patterns) without exposing individual identities
- **Data augmentation:** Oversample rare events (jackpot wins, problem gambling markers) that are heavily imbalanced in real data
- **Model testing:** Generate edge cases (maximum bet sequences, rapid session switching) that may be absent from limited real datasets
- **Fraud detection training:** SDG-GAN has been empirically validated for improving money laundering detection in online gambling by generating realistic fraudulent transaction patterns

### 7.4 Limitations

- **Distribution drift:** Synthetic data may not capture long-tail behavioural patterns (e.g., whale players, pathological chase behaviour)
- **Correlation preservation:** Multi-table relational synthesis (players → sessions → bets) remains an open research challenge
- **Mode collapse:** GAN-based methods can collapse to a subset of the real data distribution, underestimating behavioural diversity

---

## 8. Data Quality Challenges

Gambling datasets present several distinctive quality issues that practitioners must address.

### 8.1 Selection Bias

- **Survivorship bias:** Datasets only include players who remain active — those who self-exclude, go bankrupt, or are banned drop out of the record
- **Platform bias:** Behaviour on one operator's platform (e.g., bwin) does not generalise to others (e.g., DraftKings, Bet365) due to different user bases, product mixes, and regulatory environments
- **Volunteer bias:** Self-reported survey data (PGSI, gambling diaries) systematically under-represents heavy gamblers

### 8.2 Class Imbalance

Gambling classification tasks are inherently imbalanced:

- **Poker hand types:** 92% of hands are "nothing" or "one pair" across the 10-class UCI Poker Hand dataset
- **Problem gambling:** Prevalence is 1–3% of the general population, making detection a rare-class problem
- **Match fixing:** Astronomical class imbalance — fewer than 0.1% of matches show detectable manipulation

### 8.3 Temporal Drift

- **Odds market efficiency:** As sportsbooks adopt sharper modelling, historical edges decay — a model trained on 2010 soccer odds may not work in 2026
- **Regulatory changes:** US PASPA repeal (2018) transformed the sports betting landscape; pre-2018 data reflects illegal offshore markets
- **Game rule changes:** MLB pitch clock (2023), NBA play-in tournament (2020), soccer VAR adoption — all change the underlying data distribution

### 8.4 Measurement Error

- **Self-reported vs actual behaviour:** Studies show significant discrepancy between self-reported gambling deposits and actual transaction records (PMC11272738)
- **Line movement noise:** Historical odds scraped from comparison sites may differ from actual executed prices at sportsbooks
- **Missing data:** Player demographics, session end times, and cancellation reasons are frequently absent in transaction logs

### 8.5 Data Provenance

- **Scraped data:** Unofficial API scraping may violate terms of service, and data freshness is unreliable
- **Kaggle dataset staleness:** Many popular gambling datasets on Kaggle were uploaded 5–10 years ago without updates
- **Inconsistent formats:** Betting odds appear in fractional (5/1), decimal (6.0), and American (+500) formats — requiring standardisation

---

## 9. Legal & Ethical Considerations

Working with gambling data carries heightened legal and ethical obligations beyond standard data science practice.

### 9.1 Privacy Regulations

| Regulation | Jurisdiction | Key Requirements for Gambling Data |
|---|---|---|
| **GDPR** | European Union | Lawful basis for processing, data minimisation, right to erasure, DPIAs for high-risk processing |
| **UK GDPR** | United Kingdom | Same as GDPR + ICO-specific gambling guidance |
| **CCPA/CPRA** | California, USA | Right to know, right to delete, opt-out of sale |
| **LGPD** | Brazil | Similar to GDPR; consent for sensitive data |
| **Gambling Act 2005** | United Kingdom | UKGC licensing conditions, customer interaction requirements |

**GDPR-specific gambling considerations:**
- Gambling data is considered high-risk under GDPR (Article 35) — mandatory Data Protection Impact Assessment
- The EGBA Code of Conduct on Data Protection in Online Gambling provides sector-specific GDPR guidance (egba.eu)
- Scientific research exemptions (Article 89) exist but require technical safeguards — anonymisation, pseudonymisation, access controls
- Consent must be freely given, specific, and revocable — problematic for compulsive gamblers

### 9.2 Ethical Use

- **Harm amplification risk:** ML models deployed to optimise player engagement (personalised promotions, retention bonuses) can accelerate problem gambling. The AI ethics literature (Springer, 2024) identifies exploitation as a primary concern in gambling AI.
- **Duty of care:** Operators have a legal and moral obligation to identify and intervene with at-risk players. ML detection systems must be paired with appropriate intervention pathways.
- **Transparency:** Predictive models that flag "whales" or "sharps" based on behavioural data should be explainable and auditable.
- **Informed consent:** Research datasets collected by operators often have broad T&Cs that do not clearly cover downstream ML research — an evolving area of regulatory scrutiny.

### 9.3 Institutional Review & Access

- **The Transparency Project** datasets require a formal data request with research proposal and institutional affiliation
- **IRB approval** is typically required for any study involving human gambling behaviour, even with anonymised data
- **Data Transfer:** Cross-border transfer of gambling data is restricted; the EU-US Data Privacy Framework covers some transfers, but gambling-specific registrations (e.g., UK Gambling Commission) may impose additional conditions

### 9.4 Publication Restrictions

- **Kaggle and GitHub terms:** Prohibits uploading datasets containing personal gambling information. Real transaction data must be fully anonymised or synthetic.
- **Academic publishers:** Journals increasingly require data availability statements confirming ethical collection and appropriate anonymisation.
- **Commercial sensitivity:** Sportsbook odds data, even historical, is treated as proprietary IP by some operators. Always check licensing.

### 9.5 Practical Guidelines

1. **Use synthetic data** wherever possible for model development; reserve real data for final validation
2. **Aggregate to population level** (buckets, deciles, time windows) rather than individual player records
3. **Apply differential privacy** (ε ≤ 1.0) to any released statistics or trained models
4. **Never release raw behavioural traces** — session timestamps, bet sequences, and IP addresses are personally identifiable even without names
5. **Include fairness audits** — check that problem-gambling detection models do not disproportionately flag protected groups
6. **Document data provenance** — explicitly note collection methodology, bias sources, and legal basis for processing

---

## 10. Sources & Further Reading

### Primary Datasets

- [UCI Poker Hand Dataset](https://archive.ics.uci.edu/dataset/158/poker+hand) — 1M+ poker hands for classification
- [PHH Dataset (GitHub)](https://github.com/uoftcprg/phh-dataset) — 620M poker hands in standardised format
- [50 Million Blackjack Hands (Kaggle)](https://www.kaggle.com/datasets/dennisho/blackjack-hands) — Simulated blackjack rounds
- [European Soccer Database (Kaggle)](https://www.kaggle.com/datasets/hugomathien/soccer) — 25K+ matches with FIFA attributes
- [Payments Transaction Data (Data in Brief)](https://www.sciencedirect.com/science/article/pii/S2352340923001956) — Real casino/sports payment records
- [The Transparency Project](http://thetransparencyproject.org/) — Real gambling behaviour research datasets
- [NGAGE 2.0 (NCPG)](https://www.ncpgambling.org/) — US national gambling survey

### Tools & Packages

- [awesome-sports-betting (GitHub)](https://github.com/ianalloway/awesome-sports-betting) — Curated list of 100+ sports betting tools
- [sports-betting (Python)](https://georgedouzas.github.io/sports-betting/) — Dataloaders and bettors for modelling
- [CTGAN (SDV)](https://github.com/sdv-dev/CTGAN) — Synthetic tabular data generation
- [nflverse/nflfastR](https://github.com/nflverse/nflfastR) — NFL play-by-play data with built-in models
- [sportsipy](https://github.com/roclark/sportsipy) — Python wrapper for Sports Reference data

### Key Papers

- Brown & Sandholm (2019) — "Superhuman AI for multiplayer poker" (Science) — Pluribus hands dataset
- Ghaharian et al. (2023) — "Payments transaction data from online casino players" (Data in Brief, 48, 109077)
- Sandor & Bako (2024) — "Development of prediction models for problem gambling" (Greo)
- Shaffer et al. — "Population Trends in Internet Sports Gambling" (Transparency Project)
- Beating the Bookies with Their Own Numbers — arXiv 1710.02824
- A Systematic Review of ML in Sports Betting — arXiv 2410.21484v1
- AI Ethics in a Controversial Industry: The Case of Gambling — Springer AI Ethics (2024)

### Legal & Regulatory

- EGBA Code of Conduct on Data Protection in Online Gambling — [egba.eu](https://www.egba.eu/)
- GDPR and Gambling — [gdprlocal.com](https://gdprlocal.com/gdpr-compliance-online-casinos-betting-operators/)
- Data Privacy Regulations in the Gaming Industry (2025) — ScienceDirect S1930165025000135

---

*This guide was compiled from publicly available sources in July 2026. Dataset availability, licensing terms, and pricing may change. Always verify access terms with the dataset provider before use. Research involving human gambling behaviour should be reviewed by an appropriate ethics board.*
