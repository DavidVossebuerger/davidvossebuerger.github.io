---
title: "Regime-Conditional Vol-Targeting"
date: 2026-09-11
weight: 3
author: ["David Vossebürger"]
description: "Walk-forward, vol-targeted position-sizing rule combining Random Forest vol forecasts with a Gaussian HMM regime classifier and per-state vol-targets."
summary: "Walk-forward position-sizing rule with Random Forest vol forecast + Gaussian HMM regime classifier (K=3) and per-state vol-targets. 8/9 crypto beat B&H; 115/177 Russell 2000 names beat B&H."
editPost:
    URL: "https://github.com/DavidVossebuerger/Regime-Conditional-Vol-Targeting"
    Text: "Source code"

---

---

##### Overview

**Regime-Conditional Vol-Targeting (RCVT)** is a walk-forward position-sizing rule that combines a Random Forest volatility forecast with a Gaussian HMM regime classifier and per-state vol-targets. Position size is a convex mixture across regime probabilities:

```
pos(t) = Σ_k P(state=k | t) · clip(target_k / pred_vol(t+1), 0, 1)
```

Per-state targets are picked by joint brute-force over 9³ = 729 combos, optimized on walk-forward training-Sharpe with transaction costs deducted. Validation uses **block bootstrap** (60-day blocks × 1000–2000 iterations) on strict walk-forward folds; all features use `shift(+lag)` or rolling windows ending at `t` so no future information leaks.

---

##### Methodology

- **Vol forecast** — `RandomForestRegressor` on 17 lagged features: log returns, rolling std, skew, kurtosis, lags 1–5.
- **Regime classifier** — `hmmlearn.hmm.GaussianHMM` with **K=3 regimes** (calm / neutral / stress), refit per walk-forward window.
- **Per-state vol-target** — joint brute force (9³ = 729 combos), training-Sharpe with TC deducted.
- **Bootstrap validation** — 60-day blocks × 1000–2000 iters on strict walk-forward folds.
- **Lookahead hygiene** — every feature is either `shift(+lag)` or a rolling window ending at `t`; 18 unit tests include explicit lookahead-bias checks (incl. `tests/test_tc_deduction.py`).

---

##### Technical stack

- **Language:** Python 3.11+
- **Core libs:** `numpy>=1.26`, `pandas>=2.0`, `scipy>=1.11`, `scikit-learn>=1.3`, `hmmlearn>=0.3`, `matplotlib>=3.7`, `requests>=2.31`, `pyyaml>=6.0`
- **Models:** `sklearn.ensemble.RandomForestRegressor` (vol), `hmmlearn.hmm.GaussianHMM` (regime)
- **Data:** local parquet / CSV for crypto; London Strategic Edge API (`LSE_API_KEY`) for equities
- **Tests:** pytest, 18 unit tests
- **Build:** `Makefile`, `pyproject.toml` (CLI flags win, no YAML config layer)

---

##### Key results

**Crypto (9 assets, hourly bars, TC = 2 bps/side)**

- **8/9 beat B&H (89 %);** mean Δ Sharpe **+1.26**, median **+1.61**
- Best: SOL +2.53, ADA +1.96, LTC +1.78, LINK +1.68, BNB +1.61
- SOL headline weighted by the recent 2026-06-18 → 2026-09-01 window (RCVT Sharpe 6.73)
- BTC marginal (+0.03); XRP negative (−0.50)

**Russell 2000 top-200 (daily bars, TC = 10 bps/side)**

- 177 valid symbols; **115/177 beat B&H (65 %),** mean Δ Sharpe **+0.15**
- Top winners: CIFR +1.80 (P=0.94), HCC +1.24, HIMS +1.13, VSAT +1.11
- Optimistic 2 bps TC: 147/177 (83 %), mean Δ Sharpe +0.25
- Bottom-5 losers are strong Buy-and-Hold uptrends (COGT, OSCR, LUMN, PI, FLR) — typical *trend-following drag* trade-off.

**Vol-scaling experiment** (same asset × 5 vol scales, Spearman ρ = +0.287):

- Δ Sharpe plateaus around **+0.93** above 1.0× vol
- Jumps from +0.41 (0.5×) to +0.83 (1.0×)

**Drawdown cut:** Max-DD reduced 47–75 % across most crypto assets, even on underperforming names.

---

##### Source layout

```
src/multi_asset_runner.py     # crypto pipeline (hourly, 9 assets)
src/equities_runner.py       # equities pipeline (daily, Russell 2000 + LSE)
src/risk_strategy/           # extensible strategies
src/data_io/                 # asset loaders (parquet + CSV + LSE)
src/utils/                   # metrics, config loader
scripts/run_pipeline.py      # CLI entrypoint with progress + ETA
docs/architecture.md         # pipeline diagram + responsibilities
docs/results.md              # multi-asset headline + TC impact
docs/vol_scaling_results.md  # asset-independent vol scaling
docs/lse_api_notes.md        # London Strategic Edge API notes
docs/adding_new_assets.md    # extension guide
tests/                       # 18 unit tests
data/                        # input OHLC bars (gitignored)
outputs/                     # generated plots + CSVs (gitignored)
.bak/                        # legacy / superseded scripts
```

---

##### Quick start

```bash
git clone https://github.com/DavidVossebuerger/Regime-Conditional-Vol-Targeting.git
cd Regime-Conditional-Vol-Targeting
make install
make run-russell   # Russell 2000 top-200, ~10 min with cache
# or: make run-crypto   # full crypto pipeline, ~30 min
pytest tests/ -v    # 18 unit tests
```

---

##### External links

- Repo: <https://github.com/DavidVossebuerger/Regime-Conditional-Vol-Targeting>
- London Strategic Edge API docs: <https://londonstrategicedge.com/api-documentation/>
- hmmlearn docs: <https://hmmlearn.readthedocs.io/>
- scikit-learn: <https://scikit-learn.org/>
- CI: <https://github.com/DavidVossebuerger/Regime-Conditional-Vol-Targeting/actions/workflows/tests.yml>
- License: MIT
