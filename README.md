# statistical-arbitrage-engine

A fully automated statistical arbitrage system that scans a 40+ stock universe, identifies cointegrated pairs, and trades mean-reversion with regime-aware signal filtering and Monte Carlo risk analysis.

> **Ticker universe is configurable.** Built and validated on large-cap US equities. The stock list at the top of each notebook can be replaced with any universe supported by `yfinance` — Indian stocks, sector ETFs, or custom watchlists.

---

## What this does

Most pairs trading implementations hand-pick pairs manually. This system automates the entire process — from scanning hundreds of stock combinations down to the single best cointegrated pair, building the spread, generating z-score signals, backtesting, and finally stress-testing with Monte Carlo simulation. The HMM regime pipeline acts as an additional filter, ensuring trades are only taken when market conditions are favourable.

```
40+ stock universe
       │
       ▼
┌──────────────────┐
│  Correlation     │  ← screen pairs with ρ > 0.70
│  Screening       │
└──────┬───────────┘
       │ candidate pairs
       ▼
┌──────────────────┐
│  Cointegration   │  ← Engle-Granger test, p < 0.05
│  Testing         │
└──────┬───────────┘
       │ best pair (ranked by p-value)
       ▼
┌──────────────────┐
│  Spread + OLS    │  ← hedge ratio β via OLS regression
│  Hedge Ratio     │
└──────┬───────────┘
       │
       ▼
┌──────────────────┐
│  Z-Score Signal  │  ← entry ±1.6σ, exit 0.5σ
│  Generator       │
└──────┬───────────┘
       │
       ▼
┌──────────────────┐     ┌─────────────────┐
│  Backtest +      │ ──► │  HMM Regime     │
│  Equity Curve    │     │  Filter         │
└──────┬───────────┘     └─────────────────┘
       │
       ▼
┌──────────────────┐
│  Monte Carlo     │  ← 5,000 bootstrapped paths
│  Risk Analysis   │     VaR, loss probability, fan chart
└──────────────────┘
```

---

## Project structure

```
statistical-arbitrage-engine/
│
├── notebooks/
│   ├── 01_QuantPairs_Engine.ipynb       # Full pairs scanning + trading system
│   └── 02_HMM_Regime_Pipeline.ipynb    # Regime detection as signal filter
│
├── research/
│   └── Statistical_Arbitrage_Pairs_Trading.pdf
│   └── HMM_Market_Regime_Detection.pdf
│
├── requirements.txt
└── README.md
```

---

## Notebooks

### 1. QuantPairs Engine (`01_QuantPairs_Engine.ipynb`)

The core of this project. Fully automated pipeline from raw data to trade signals to risk analysis.

**Step 1 — Pair scanning**

Downloads price history for 40+ large-cap US stocks across 7 sectors (tech, finance, energy, consumer, healthcare, payments, industrials). Computes all pairwise Pearson correlations and shortlists pairs with ρ > 0.70.

**Step 2 — Cointegration testing**

Applies the Engle-Granger two-step test on all shortlisted pairs:
```
Y_t = α + βX_t + ε_t
ADF test on residuals ε_t → if stationary, pair is cointegrated
```
Pairs with p-value < 0.05 are selected. The best pair is ranked by p-value.

**Step 3 — Spread construction**

OLS regression estimates the fixed hedge ratio β:
```
spread_t = Y_t − β × X_t
```
The spread is modelled as an Ornstein-Uhlenbeck mean-reverting process.

**Step 4 — Z-score signals**

```
Z_t = (spread_t − μ_rolling) / σ_rolling    [60-day window]

Z < −1.6  →  LONG spread  (buy Y, sell βX)
Z > +1.6  →  SHORT spread (sell Y, buy βX)
|Z| < 0.5 →  EXIT
```

**Step 5 — Backtested results (V-MA pair)**

| Metric | Value |
|--------|-------|
| Sharpe Ratio | ~0.75 |
| Total Return | ~140% |
| Max Drawdown | ~18% |
| Win Rate | ~58% |

**Step 6 — Monte Carlo risk analysis**

5,000 bootstrapped equity paths over 252 days:

| Scenario | Outcome |
|----------|---------|
| Worst case (5th pct) | ~0.90× |
| Median | ~1.08× |
| Best case (95th pct) | ~1.32× |
| Loss probability | ~24% |

Outputs both a professional confidence-band chart and a full rainbow fan chart.

---

### 2. HMM Regime Pipeline (`02_HMM_Regime_Pipeline.ipynb`)

A complete Hidden Markov Model pipeline that detects market regimes (Bull / Bear / Sideways) and can be used to filter pairs trading signals — only entering trades when the HMM confirms a regime favourable to mean reversion (typically the sideways / low-volatility state).

See [`adaptive-regime-ai`](https://github.com/yourusername/adaptive-regime-ai) for the full regime + LSTM system this notebook is part of.

---

## Quickstart

```bash
git clone https://github.com/yourusername/statistical-arbitrage-engine
cd statistical-arbitrage-engine
pip install -r requirements.txt
```

Open `01_QuantPairs_Engine.ipynb` — all data downloads automatically. The pair scanner will run through the full universe and select the best cointegrated pair automatically. No manual pair selection needed.

To use a different stock universe, edit the `stocks` list at the top of the notebook.

---

## Requirements

```
yfinance
numpy
pandas
matplotlib
statsmodels
hmmlearn
scikit-learn
```

---

## Key concepts

| Concept | Used in |
|---------|---------|
| Pearson correlation | Pair pre-screening |
| Engle-Granger cointegration | Pair selection |
| OLS regression | Hedge ratio estimation |
| Ornstein-Uhlenbeck process | Spread mean-reversion model |
| Z-score normalization | Entry/exit signal generation |
| Bootstrap Monte Carlo | Forward risk simulation |
| Hidden Markov Models | Regime-based signal filtering |

---

## Limitations & honest notes

- Fixed hedge ratio (β) can drift over time — rolling or Kalman filter estimates would be more robust.
- Cointegration relationships can break during structural market shifts.
- No transaction costs or slippage modelled.
- Single-pair system — a real stat-arb desk runs a portfolio of pairs simultaneously.
- Walk-forward validation not yet implemented.

---

## What's next

- [ ] Kalman filter for dynamic hedge ratio
- [ ] Multi-pair portfolio with correlation-adjusted position sizing
- [ ] Regime-conditional entry — only trade in HMM sideways state
- [ ] High-frequency data extension

---

## Author

**Piyush Patil** — AI & Data Science / Quantitative Finance  
Built as part of a personal quantitative research portfolio covering Indian and US equity markets.
