# Beating the Book: NBA (and Football) Betting Notebooks

This project contains Jupyter notebooks and data files I use to find positive expected value (EV) bets and manage volatility by placing multiple uncorrelated wagers at the same time. The approach has worked historically on basketball and football; the current repo focuses on NBA but the workflow generalizes to other sports/leagues.

> Disclaimer: This is for educational purposes only. Sports betting involves risk. Nothing here is financial advice. Respect your local laws and each data provider’s Terms of Service.

---

## What’s inside

- `Player_Stats.ipynb`:
  - Pulls NBA odds and markets from sportsbooks (primarily Unibet endpoints) and writes them to CSVs.
  - Builds features around player and team statistics (e.g., points/rebounds/assists combos) and compares them to offered lines.
  - Helps surface edges by contrasting model-driven expectations with market odds.

- `RegLin.ipynb`:
  - Exploratory modeling and regression/visualization to estimate fair probabilities/lines from historical data and current team context.
  - Loads consolidated odds (e.g., `Odds_NBA.csv`) and team/player datasets to quantify value vs. the market.

- `GetMacroInfo.ipynb`:
  - Gathers macro/contextual information that can affect games (schedule, aggregates, basic trends) to enrich the feature set.

- Data files:
  - `Odds_Match.csv`: Moneyline markets (columns like `id, Match, Team, OddWin, Date`).
  - `Odds_NBA.csv`: Player props and combos with decimal odds for over/under variants (e.g., `PTS_REB_AST`, `PTS_REB`, `PTS_AST`, `REB_AST`, plus `Odd+...` and `Odd-...`).
  - `Players_NBA.csv`: Player metadata (name, team, position, age, height, salary, ESPN URL, etc.).
  - `Teams_NBA.csv`: Team-level metrics split by home/away (e.g., points, rebounds, assists, turnovers).

---

## How it works (end-to-end)

1. Update data
   - Run `Player_Stats.ipynb` to fetch current odds and write/update `Odds_Match.csv` and `Odds_NBA.csv`.
   - Player and team reference data live in `Players_NBA.csv` and `Teams_NBA.csv`.

2. Estimate fair probabilities/lines
   - Use `RegLin.ipynb` to fit simple models and calibrations (e.g., linear relationships, moving averages, contextual adjustments).
   - Convert model outputs to fair probabilities or fair prices/lines for each market.

3. Find edges
   - Compare fair prices to actual odds to flag positive EV opportunities.
   - Example (decimal odds `o`, fair probability `p`): expected value per unit stake is `EV = p * (o - 1) - (1 - p)`.

4. Size stakes and diversify
   - Bet multiple independent/low-correlated markets in parallel to reduce variance.
   - Use conservative fractional Kelly sizing or fixed-percentage stakes to control drawdowns (more below).

5. Execute and track
   - Place bets, then log results back into the notebook/data store to continuously evaluate calibration and edge quality.

---

## Bankroll, volatility, and staking

- Diversification
  - Placing N uncorrelated bets simultaneously reduces volatility roughly with \(\sim 1/\sqrt{N}\). Favor many small independent edges over one large bet.

- Fractional Kelly (recommended)
  - For decimal odds `o`, define \(b = o - 1\). With fair win probability \(p\), \(q = 1 - p\), full Kelly fraction is:
    \[ f^* = \frac{bp - q}{b} \]
  - Stake a fraction of Kelly: \(f = k \cdot f^*\), with \(k \in [0.1, 0.5]\) typical. Cap stakes to avoid concentration.

- Simple fixed-percentage alternative
  - Stake 0.25%–1.0% of bankroll per bet depending on edge magnitude and correlation. Lower if correlation is high.

- Practical tips
  - Prioritize edges with robust signal (multiple features agree, stable across sensitivity checks).
  - Limit exposure to highly correlated markets (e.g., overlapping player props on the same game).
  - Maintain a running log of realized outcomes to iterate on calibration.

---

## Odds and data sources

- Sportsbook odds
  - Primary data currently queries Unibet JSON endpoints like `zones/v3/sportnode/markets.json` with specific `nodeId`s and filters for basketball markets.
  - Node IDs and market names can change. If you stop seeing data, re-check the sportsbook UI/API to refresh IDs and filters.
  - Always respect the sportsbook’s robots.txt/ToS; throttle requests and cache when possible.

- Player/team data
  - `Players_NBA.csv` and `Teams_NBA.csv` are curated snapshots that notebooks join with odds.
  - You can refresh them from public sources (e.g., ESPN pages) if the schema remains consistent.

---

## Running the notebooks

1. Environment
   - Python 3.9+ recommended.
   - Install packages:
     ```bash
     pip install pandas numpy requests beautifulsoup4 selenium webdriver-manager seaborn matplotlib jupyter
     ```
   - Selenium notes: `webdriver-manager` auto-installs ChromeDriver; ensure Chrome/Chromium is available on your machine.

2. Launch and run
   - Start Jupyter and open the notebooks:
     ```bash
     jupyter notebook
     ```
   - Suggested order: run `Player_Stats.ipynb` (to refresh odds/data), then `RegLin.ipynb` (to evaluate edges). Use `GetMacroInfo.ipynb` as needed for contextual features.

3. Customize for football or other leagues
   - Swap endpoints/filters for the sportsbook API to target different competitions.
   - Update joins to the appropriate player/team reference files and re-run the same edge-finding workflow.

---

## Limitations and next steps

- Lines move quickly; latency between fetch and bet placement can erode edges.
- Market vigorish skews implied probabilities; ensure you de-vig correctly, especially in multi-outcome markets.
- Models in `RegLin.ipynb` are intentionally simple; consider upgrading to calibrated classification/regression, backtesting with walk-forward splits, and correlation-aware portfolio construction.
- Consider logging all candidate edges, placed bets, and PnL to a database or versioned CSV for proper monitoring.

---

## Ethics and compliance

- Comply with local regulations and operator ToS.
- Do not scrape aggressively; respect rate limits and cache responses.
- Use this repository at your own risk.
