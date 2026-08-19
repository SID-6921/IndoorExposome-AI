# FROZEN configuration — portable cross-dataset model

This is a **separate, distinct frozen artifact** from the original primary model (`FROZEN_CONFIG.md`). Same model family, reduced feature set — do not confuse the two or imply the original frozen model was reused unchanged.

## Why a separate model was needed

The original frozen model uses the "Full+" feature set (groups A–E), which includes outdoor PM2.5 (group B) and indoor CO2 (part of group C) — neither exists in the 100-Air-Purifiers dataset. The input vector would not match, so the original artifact cannot be run on this data as-is.

## Resolution mismatch: LBNL (1-min) vs. 100-Air-Purifiers (hourly)

Rather than awkwardly translate minute-scale windows onto an hourly series, **LBNL was aggregated to hourly means first**, and the same hourly-scale feature construction was then applied identically to both datasets — this is the only way to guarantee the trained coefficients mean the same thing on both sides.

## Group F (purifier on/off, airflow) — excluded from the trained model

Kolliputi's plan proposed a new "group F" (purifier power state + airflow) analogous to the old group D (activity/equipment). **This was not included as a model input.** LBNL has no equivalent signal to train a coefficient against — homes there ran continuous central mechanical ventilation per protocol (no on/off variability to learn from), so there is nothing to map group F onto during LBNL training. Including it would mean the coefficient is either untrained (arbitrary) or the model would need to be trained directly on the 100-homes data, which would violate the "pure external test, no training on this data" design Kolliputi himself set. Group F fields were computed and retained in the data (`F_purifier_on`, `F_airflow`) for potential future exploratory stratification, but did not feed the frozen model.

## Episode definition (hourly-resolution adaptation)

The original ≥15-consecutive-minute sustained-duration rule cannot be evaluated from hourly averages. Adapted definition, applied identically to both datasets: **PM2.5 ≥35 µg/m³, sustained for ≥1 consecutive hour** (the finest resolvable duration at this resolution). Threshold unchanged at 35 µg/m³ (in the 100-homes pooled distribution this sits at ≈p95.7, slightly less extreme than LBNL's p97.5 but still a genuine tail event, not renormalized to force a match).

## Feature set (identical construction on both datasets)

- Current PM2.5; lags at 1h/2h/3h/6h; rolling mean/max/std at 3h/6h/12h; 3-hour rate of change; hours since PM2.5 last crossed 35 µg/m³ (capped at 72h).
- Current indoor temperature; current relative humidity.
- All features strictly causal.

## Preprocessing, model, training

`SimpleImputer(median)` + `StandardScaler()`, fit on the LBNL hourly-aggregated 59-home pool only. `LogisticRegression(max_iter=2000, class_weight="balanced")` — same model family and parameters as the original frozen model. Trained on all 59 LBNL CV-pool homes (hourly-aggregated).

## Operating threshold

**0.4121270631400635** — selected via 5-fold group-CV on the LBNL hourly-aggregated dev pool only (≤2 false-alerts/home-day point, 3-hour cooldown). Never re-selected on the 100-homes external data.

## Horizons

Primary: 3 hours (gives ≥3 lead samples at hourly resolution, satisfying the ≥3x-horizon rule). Secondary/sensitivity: 2 hours (below the rule of thumb — reported transparently as a secondary number only, per instruction).

## Evaluation

All 100 external homes, evaluated once, no training/tuning/threshold re-selection on this data at any point.
