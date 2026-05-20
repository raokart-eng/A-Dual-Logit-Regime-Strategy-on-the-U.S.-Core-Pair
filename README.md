# Karthik-1 Strategy — Notebook Bundle

**Team 6 · FE747 Quantitative Investment Strategies · Spring 2026**

A self-contained, reproducible implementation of the Karthik-1 dual-logit regime strategy. Open the notebook, set one path, and every artifact you need is regenerated from raw data.

---

## Quick start

1. Install dependencies:
   ```bash
   pip install -r requirements.txt
   ```

2. Open `Karthik1_Strategy.ipynb` and confirm that the `DATA_DIR` cell points at the bundled `data/` folder:
   ```python
   DATA_DIR = Path("./data")
   OUT_DIR  = Path("./karthik1_outputs")
   ```

3. Run all cells (~30 seconds). Charts render inline; export files appear in `karthik1_outputs/`.

That is the entire workflow. Everything else in this bundle is supporting material.

---

## What's in this bundle

```
Karthik1_Strategy_Bundle/
├── Karthik1_Strategy.ipynb         <- the main deliverable, runs end-to-end
├── README.md                       <- this file
├── requirements.txt                <- Python dependencies
│
├── data/                           <- INPUT: six daily price CSVs (1990-2026)
│   ├── VFINX_daily.csv             (Vanguard 500 Index)
│   ├── VEXMX_daily.csv             (Vanguard Extended Market)
│   ├── VWIGX_daily.csv             (Vanguard International Growth)
│   ├── VBMFX_daily.csv             (Vanguard Total Bond Market)
│   ├── VUSTX_daily.csv             (Vanguard Long-Term Treasury)
│   └── VWEHX_daily.csv             (Vanguard High-Yield Corporate)
│
├── models/                         <- OUTPUT: fitted model artifacts
│   ├── Karthik-1_drawdown_model.json
│   ├── Karthik-1_upside_model.json
│   └── Karthik-1_strategy_config.json
│
├── probabilities/                  <- OUTPUT: daily P(DD) and P(UP) for full panel
│   ├── Karthik-1_drawdown_probs.csv
│   └── Karthik-1_upside_probs.csv
│
├── simulations/                    <- OUTPUT: full daily strategy simulation
│   └── Karthik-1_daily_simulation.csv
│
├── diagnostics/                    <- OUTPUT: charts produced by the notebook
│   ├── 01_dd_logit_coefficients.png
│   ├── 02_up_logit_coefficients.png
│   ├── 03_dd_confusion_matrices_3panel.png
│   ├── 04_up_confusion_matrices_3panel.png
│   ├── 05_showcase_window_equity_curves.png
│   ├── 06_adversarial_window_equity_curves.png
│   └── 07_state_attribution_quarterly.png
│
└── extras_analysis/                <- OPTIONAL: supporting analyses (not required to run notebook)
    ├── competition_phase1_grid.csv
    ├── competition_phase2_alloc_opt.csv
    ├── competition_phase3_validation.csv
    ├── competition_winner.json
    ├── dd_threshold_grid.csv
    ├── up_threshold_grid.csv
    ├── threshold_sweep_fixed.csv
    ├── best_4yr_windows_VFINX_VWEHX.csv
    ├── best_window_DD_UP_optim.csv
    ├── rolling_windows_full_history.csv
    ├── random_windows_full_history.csv
    └── 2021_2024_state_attribution.csv
```

---

## File reference

### The notebook

**`Karthik1_Strategy.ipynb`** — 33 cells, runs end-to-end in ~30 seconds. Five logical sections:

1. **Configuration** (cells 1-3): all hyperparameters in one `CONFIG` dict
2. **Data and features** (cells 4-9): load CSVs, build 21-feature panel, construct DD and UP targets
3. **Modeling** (cells 10-13): fit two L1 logits, plot coefficients, predict on full panel
4. **Diagnostics** (cells 14-19): confusion matrices, AUC table across IS / Pre-IS / Post-IS
5. **Simulation and exports** (cells 20-32): four-state allocation engine, performance tables, equity curves, file exports

### The data (`data/`)

Six Vanguard mutual funds with continuous daily history from 1990-01-02. Two columns each: `Date`, `Close` (adjusted close).

| Ticker | Type | Role in strategy |
|---|---|---|
| VFINX | Equity | **Selected equity asset** (S&P 500 index fund) |
| VEXMX | Equity | Used in `SMB_proxy` feature (extended market vs S&P) |
| VWIGX | Equity | Used in `INTL_US` feature (international vs US) |
| VBMFX | Fixed Income | **Selected fixed income asset** (total bond market) |
| VUSTX | Fixed Income | Used in `Term` feature (long Treasury vs aggregate bond) |
| VWEHX | Fixed Income | Used in `Credit` feature (high yield vs Treasury) |

### The models (`models/`)

**`Karthik-1_drawdown_model.json`** — fitted L1 logit predicting P(forward 63d max drawdown ≥ 15%). Contains intercept + 21 coefficients + the 0.40 probability cutoff used to convert prob → binary signal.

**`Karthik-1_upside_model.json`** — same shape, predicts P(forward 63d max return ≥ 8%) with cutoff 0.30.

**`Karthik-1_strategy_config.json`** — asset pair, four-state allocations (90/55/15/85), monthly rebalance flag, challenge window assignment.

These three JSONs are the format the FE747 sim engine consumes.

### The probabilities (`probabilities/`)

Daily probability series exported across the full available panel (1991-01-30 → 2026-01-20, ≈ 8,800 rows each). Two columns: `Date`, `P_DD` (or `P_UP`). Re-load these instead of refitting if you want to skip the training step.

### The simulation (`simulations/`)

**`Karthik-1_daily_simulation.csv`** — the canonical strategy output. One row per trading day with columns:

| Column | Meaning |
|---|---|
| `date` | trading day |
| `p_dd`, `p_up` | model probabilities for that day |
| `signal_dd`, `signal_up` | binary signals after applying cutoffs |
| `state` | active state for that day (AGG / NEUT / CONF / DEF) — already accounts for monthly rebalance |
| `alloc_eq` | equity weight in force that day (90% / 55% / 15% / 85%) |
| `eq_ret`, `fi_ret` | underlying VFINX and VBMFX simple returns |
| `strat_ret` | strategy daily return |
| `bmk_ret` | static 50/50 benchmark daily return |
| `cum_strat`, `cum_bmk` | cumulative wealth indices (start = 1.0) |
| `rel_wealth` | strategy wealth − benchmark wealth (the chart shown on Slide 12) |

To compute performance for any window, slice this CSV by date.

### The diagnostics (`diagnostics/`)

Seven PNG charts generated by the notebook. Drop them into the deck or process doc as exhibits.

| File | Slide reference |
|---|---|
| `01_dd_logit_coefficients.png` | §2.1 of process doc, Slide 5 |
| `02_up_logit_coefficients.png` | §2.1 of process doc, Slide 7 |
| `03_dd_confusion_matrices_3panel.png` | Slide 13 of deck |
| `04_up_confusion_matrices_3panel.png` | Slide 13 of deck |
| `05_showcase_window_equity_curves.png` | Reference for showcase window claims |
| `06_adversarial_window_equity_curves.png` | Slide 12 of deck |
| `07_state_attribution_quarterly.png` | Slide 14 of deck |

### Supporting analyses (`extras_analysis/`)

These are separate analyses that informed the strategy's design choices but are not required to run the notebook. They back up specific claims made in the deck and process document.

| File | What it documents |
|---|---|
| `competition_phase1_grid.csv` | Coarse grid search across 9 pairs × 5 DD% × 4 UP% × 16 cutoff combos |
| `competition_phase2_alloc_opt.csv` | Allocation re-optimization (DE search) for top 10 candidates |
| `competition_phase3_validation.csv` | 500 fresh random 4-yr windows for top 5 finalists |
| `competition_winner.json` | The winning configuration after all three phases |
| `dd_threshold_grid.csv`, `up_threshold_grid.csv` | Joint sweep of event thresholds × probability cutoffs |
| `threshold_sweep_fixed.csv` | UP-only threshold sensitivity at fixed cutoffs |
| `best_4yr_windows_VFINX_VWEHX.csv` | Performance ranking of all 33 rolling 4-yr windows |
| `best_window_DD_UP_optim.csv` | DD%/UP% re-tuning for the chosen showcase window |
| `rolling_windows_full_history.csv` | Strategy performance across 33 rolling 4-yr windows, 1991-2026 |
| `random_windows_full_history.csv` | 500 random 4-yr windows for robustness validation |
| `2021_2024_state_attribution.csv` | Daily state and allocation series for the adversarial window |

Each row is documented in `extras_analysis_README.md` (one line per file describing the columns).

---

## How the pieces connect

```
                  data/*.csv   ──┐
                                 │
                                 ▼
                       Karthik1_Strategy.ipynb
                                 │
                  ┌──────────────┼──────────────┐
                  │              │              │
                  ▼              ▼              ▼
         models/*.json   probabilities/*.csv   simulations/*.csv
                  │              │              │
                  └──────────────┼──────────────┘
                                 ▼
                          diagnostics/*.png
```

**One pass through the notebook regenerates everything in `models/`, `probabilities/`, `simulations/`, and `diagnostics/`.** The `data/` folder is the only thing the notebook reads from. The `extras_analysis/` folder is independent — those CSVs were produced by separate scripts during the design phase.

---

## Reproducing the headline numbers

After running the notebook, these numbers should appear in cell 26's performance table:

| Window | Strategy CAGR | Benchmark CAGR | Alpha |
|---|---|---|---|
| Showcase 1995-1998 (in-sample) | +23.24% | +20.30% | +2.94% |
| Pre-IS 1991-1994 (true OOS) | +13.18% | +9.91% | +3.27% |
| Adversarial 2021-2024 | +6.99% | +5.68% | +1.31% |
| Full panel | +9.53% | +8.50% | +1.03% |

Year-by-year breakdown for 2021-2024 (cell 26):

| Year | Strategy | Benchmark | Alpha |
|---|---|---|---|
| 2021 | +15.99% | +12.77% | +3.22% |
| 2022 | −16.38% | −15.23% | −1.15% |
| 2023 | +19.55% | +15.84% | +3.72% |
| 2024 | +12.99% | +12.61% | +0.37% |

If you don't see these numbers, something is wrong — most likely `DATA_DIR` is pointing at the wrong folder.

---

## Honest caveats (also in cell 33)

1. **In-sample DD recall (100%) is an overfit artifact.** Out-of-sample recall is 47.9% on 6,767 unseen days. The strategy's drawdown protection comes from the 85% equity weight in the Defensive state, not from signal accuracy.

2. **The Mom_21d coefficient of ~+1500 is unusually large.** L1 with C=1.0 didn't fully constrain the model. C=0.1 would produce more interpretable coefficients but might reduce in-sample fit.

3. **The Upside model is barely better than random** at the 8% threshold (in-sample accuracy 57.6% vs base rate 56.5%). It functions as a "default on" filter, not a discriminative classifier.

4. **No transaction costs.** Real deployment with monthly rebalancing would face ~1-2% annual frictional drag.

5. **The 1995-1998 fit window over-represents fast V-shaped crashes.** The model has never seen a slow grinding bear like 2000-02 or 2008-09, which is why DD recall degrades severely on those periods OOS.

The most impactful v2 change would be retraining on 2002-2018 (five distinct crash regimes vs two). Smaller showcase numbers, much better generalization.

---

## Contact

Team 6, FE747 Spring 2026, Boston University Questrom School of Business.
