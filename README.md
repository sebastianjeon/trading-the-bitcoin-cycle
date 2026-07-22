# Trading the Bitcoin Cycle

**Does Bitcoin's position within its market cycle predict returns better than the technical and macroeconomic indicators most models rely on?**

Four XGBoost specifications, one expanding-window walk-forward design, only the feature set changes. Cycle-position features won on accuracy — and the specification with the *highest* accuracy produced the *worst* risk-adjusted returns.

📄 **[Full paper (46 pages)](./paper/Trading_the_Bitcoin_Cycle - Sebastian Jeon.pdf)**

---

## Headline results

| Model | OOS accuracy | Macro F1 |
|---|---|---|
| Technical (23 features) | 28.4% | 0.193 |
| Macroeconomic (7 features) | 40.3% | 0.314 |
| **Cycle (8 features)** | **47.6%** | **0.393** |
| Combined (38 features) | 37.4% | 0.278 |

Finalized ±3% specification, pooled 2021–2025: **51.26%** accuracy, **0.401** Macro F1, log loss **1.500** (vs 1.762 for the macro benchmark). Diebold–Mariano 5.875 (p < 1e-6); McNemar 43.75 (p = 3.7e-11).

### Long/Cash strategy, net of 10bps per unit of turnover

| | Buy & hold | Long/Cash |
|---|---|---|
| CAGR | 26.13% | **62.07%** |
| Sharpe | 0.69 | **1.31** |
| Max drawdown | −81.53% | **−51.86%** |
| Market exposure | 100% | **48.51%** |

Bootstrap (5,000 moving-block samples): Sharpe beat buy-and-hold in 98.6% of samples (p = 0.014), drawdown in 99.3% (p = 0.007). CAGR advantage appeared in 92.2% but did not clear 5% significance.

---

## The finding that matters

Labeling every move above 0% as bullish gave the **highest classification accuracy of any specification tested (55.7%)** and the **worst trading performance (Sharpe 0.77)**. A less accurate specification returned 1.31.

Most of the additional "correct" predictions were sub-1% moves whose edge was smaller than the cost of trading them.

| Threshold | OOS accuracy | CAGR | Sharpe |
|---|---|---|---|
| 0% | **55.73%** | 26.09% | 0.765 |
| **3%** | 51.11% | **62.07%** | **1.310** |
| 5% | 46.08% | 56.31% | 1.249 |
| 10% | 44.57% | 30.84% | 0.875 |

Classification accuracy and economic value are different objectives.

---

## Honest limitations

This is research output, not a deployable system.

- **Look-ahead in the cycle features.** `Days_Since_Bottom`, `Days_Until_Top`, `Percent_Through_Cycle`, `Bull_Progress`, and `Bear_Progress` are measured from market bottoms identifiable only in hindsight. A halving-only specification using strictly ex-ante observable dates achieves Sharpe 0.82 — the honest floor.
- **Few independent regimes.** ~3,400 daily rows, but only three completed cycles. Effective sample size is far smaller than the row count suggests.
- **Overlapping targets.** Forecasts one day apart share 29 of 30 forward return days. Moving-block bootstrap partially addresses this; standard tests would overstate precision.
- **Costs are a flat 10bps.** No slippage, financing, taxes, short-borrow, or cash yield modeled.
- **Macro revision bias.** Current FRED vintages are used, not archived real-time vintages.

---

## Repository

```
├── notebooks/
│   └── btc_cycle_model.ipynb      # full pipeline: features → walk-forward → backtest
├── paper/
│   └── Trading_the_Bitcoin_Cycle.pdf
├── requirements.txt
└── README.md
```

## Reproducing

```bash
git clone https://github.com/<sebastianjeon>/trading-the-bitcoin-cycle.git
cd trading-the-bitcoin-cycle
pip install -r requirements.txt
jupyter lab notebooks/btc_cycle_model.ipynb
```

Data is pulled at runtime — no keys required.

| Source | Series |
|---|---|
| Yahoo Finance (`yfinance`) | BTC-USD, DX-Y.NYB |
| FRED (`pandas-datareader`) | FEDFUNDS, CPIAUCSL, M2SL, DGS10, SP500, NASDAQCOM |

Prices update continuously, so exact figures will drift from those reported in the paper. The sample used there ends **2026-06-14**.

## Method

- **Target** — 30-day forward return, three classes at ±3% (Bearish / Neutral / Bullish)
- **Model** — XGBoost, 300 trees, depth 4, lr 0.05, 0.8 subsampling, seed 42
- **Validation** — expanding-window walk-forward; each test year predicted from earlier years only
- **Backtest** — signals shifted +1 day; 0.10% per unit of turnover

## Citation

```bibtex
@techreport{jeon2026btccycle,
  title  = {Trading the Bitcoin Cycle: Evaluating Technical, Macro, and
            Cycle-Based Signals in a Walk-Forward Framework},
  author = {Jeon, Sebastian},
  year   = {2026},
  type   = {Working paper},
  institution = {ISLab, Seoul National University}
}
```

## License

MIT for code. The paper is © 2026 Sebastian Jeon, all rights reserved.
