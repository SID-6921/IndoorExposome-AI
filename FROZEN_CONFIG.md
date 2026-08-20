# FROZEN configuration: primary model, locked before opening the 10-home LBNL holdout

Written and committed before any holdout data is touched. Nothing below changes after seeing holdout results, per PI instruction.

## Episode definition
- Indoor PM2.5 ≥ 35 µg/m³, sustained ≥15 consecutive minutes (1-min grid, NaN breaks a run).
- Primary horizon: 45 minutes. Secondary: 30 and 60 minutes (reported, not promoted).

## Feature set ("Full+" = groups A+B+C+D+E, exact columns from `step2_pipeline.py::build_features_and_labels`)
- **A (indoor PM history):** current value; lags at 5/10/15/30/60 min; rolling mean/max/std at 15/30/60 min windows; 15-min rate of change; minutes since PM last crossed 35 µg/m³ (capped at 180).
- **B (+outdoor):** outdoor PM smoothed via 10-min trailing rolling mean (causal, min_periods=5); smoothed value lagged 15 min; 30-min rolling mean; indoor-minus-smoothed-outdoor difference.
- **C (+environmental):** indoor CO2 current + 30-min rolling mean; indoor temperature current; indoor relative humidity current.
- **D (+activity):** max cooktop-burner temperature current; 30-min rolling max of cooktop temperature.
- **E (hypothesis-driven pass):** 30-min PM slope; PM acceleration (change in 15-min slope); CO2 15-min slope; RH 15-min change; cooktop-temperature 15-min change; minutes since burner activation (cooktop >5°C above its own 60-min rolling median, capped at 180); cooking×CO2 interaction; cooking×low-ventilation interaction.
- All features are strictly causal (use only data at or before the prediction minute).

## Preprocessing
- `SimpleImputer(strategy="median")`, fit on the 59-home training pool only, applied unchanged to holdout.
- `StandardScaler()`, fit on the 59-home training pool only, applied unchanged to holdout.

## Model
- `LogisticRegression(max_iter=2000, class_weight="balanced")`, scikit-learn defaults otherwise (solver="lbfgs", deterministic, no random_state needed).
- Trained on **all 59 CV-pool homes** (not fold-restricted: the 5-fold CV was for validation/threshold-selection only, and the deployed model for the holdout is refit on the full development pool).

## Operating point (probability threshold)
- **0.8711552648952829**, the threshold that produced ≤2.0 false alerts/home-day on the 59-home CV pool via 5-fold out-of-fold evaluation (Step 2B). Selected entirely on development data. Not re-selected on holdout.

## Alert consolidation rule
- 30-minute cooldown: once an alert fires for a home, no new alert counts for that home until 30 minutes have elapsed, regardless of intervening probability values.
- An alert is a "hit" if the per-minute label at its timestamp is 1 (an episode onset falls within the next 45 minutes); otherwise a "false alert."

## Evaluation metrics (holdout, computed once)
Number of homes; number of qualifying episodes; event sensitivity; distinct false alerts/home-day; alert precision; median/mean first-alert lead time (+ IQR); % of episodes warned ≥10/20/30 min early; AUPRC; AUROC; bootstrap confidence intervals (home-level cluster resampling, 2000 iterations, since homes, not minutes, are the independent unit).

## Rule
No threshold change, no feature change, no recalibration, no retraining, no model reselection after this file is written. If the primary result is unfavorable, it is reported directly.
