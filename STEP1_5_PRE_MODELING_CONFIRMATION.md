# IndoorExposome AI - Pre-Modeling Confirmation Report

## 1. LBNL dataset: sampling interval, usable homes, sync, missingness, duration

- Sampling interval is confirmed at 1 minute (60s) across all 69 homes with usable data, measured directly from timestamp deltas in the raw per-home CSVs rather than assumed from the README.
- 69 of 70 homes are usable. House 006 has no indoor PM2.5 channel at all (instrument not deployed or failed) and was dropped.
- Synchronized indoor+outdoor PM2.5 observations total 614,030 minute-pairs (both non-null at the same timestamp), across 67 of the 69 homes. Two homes have indoor PM data but zero valid outdoor pairs (outdoor sensor failure).
- Missingness averages 5.17% indoor and 2.56% outdoor (per-home average). Total valid indoor readings: 628,647.
- Usable monitoring duration after cleaning is 460.5 home-days (mean 6.67 days/home, range 3.62–7.65 days), consistent with the ~1-week protocol.
- A data-quality note from the LBNL README, not previously flagged: *"The outdoor PM time series is noisy at 1-minute resolution... recommended to average over at least 10 minutes."* This doesn't affect the horizon, but it means outdoor PM2.5 should enter the model as a ≥10-min rolling mean rather than raw 1-min values. The PM sensor is a MetOne photometer (light-scattering, µg/m³), and a documented ~0.1% raw transcription error rate was found and corrected by the original authors.

## 2. Forecasting horizon

**30–60 min horizon is retained.** At 1-min resolution, a 45-min horizon corresponds to 45 lead samples, well above the ≥3x-horizon rule of thumb. No change needed to the original Step 1 proposal.

## 3. Episode definition: tested against the actual PM2.5 distribution, not assumed

Pooled distribution across all 628,647 valid indoor readings (highly right-skewed): p50=2, p75=5, p90=11, p95=19, **p97.5=33**, p99=67, mean=6.17, max=2,006 µg/m³. Per-home baselines are very heterogeneous: home medians range 0–16 µg/m³, home p95s range 1–189 µg/m³, so a single fixed absolute threshold will systematically favor already-dirty homes.

Five candidate definitions tested (min duration 15 min, 45-min horizon unless noted):

| Definition | Threshold basis | Episodes | Homes w/ ≥1 event | Positive windows | Class balance |
|---|---|---|---|---|---|
| **EPA 24h-NAAQS proxy** | 35 µg/m³ (≈p97.5 of actual data) | 140 | 35/69 (51%) | 0.977% | 1:101 |
| Lower absolute | 25 µg/m³ (≈p97 actual) | 172 | 40/69 (58%) | 1.207% | 1:82 |
| WHO 24h-guideline proxy | 15 µg/m³ (≈p90 actual) | 285 | 50/69 (72%) | 1.961% | 1:50 |
| Per-home relative, p95 (floor 12) | adapts to each home's own baseline | 231 | 55/69 (80%) | 1.602% | 1:61 |
| Per-home relative, p90 (floor 12) | adapts to each home's own baseline | 286 | 55/69 (80%) | 1.956% | 1:50 |

35 µg/m³ isn't just the familiar regulatory number here: it independently lands at ~p97.5 of the real pooled distribution, a tail event in this data, which is the empirical justification you asked for. 15 µg/m³ (WHO) gives much better class balance and home coverage but is a chronic 24-hr/annual guideline, not an acute-episode threshold, which is weaker scientific justification for this use even though statistically convenient.

**Recommendation:**
- **Primary:** 35 µg/m³, ≥15 min, pooled absolute threshold (empirically ≈p97.5, matches original Step 1 proposal).
- **Sensitivity analysis:** per-home relative threshold (p90, 12 µg/m³ floor); corrects for the large inter-home baseline heterogeneity, nearly doubles usable homes (55 vs 35) and episodes, and keeps a floor so ultra-clean homes' noise doesn't count as an "episode."
- Flag clearly: under the primary definition, positive-class prevalence is ~1% (1:101), a real, expected constraint for a rare-event forecasting task that should be handled with class weighting / AUPRC-first reporting / calibration analysis, not treated as a bug.

## 4. Validation architecture

Home-level group separation confirmed feasible: with 69 usable homes, propose holding out ~10 homes (~15%) as a final untouched LBNL test set, with group k-fold (by home) across the remaining ~59 for model selection/tuning. No timestamp-based splitting anywhere. The 100 Air Purifiers dataset stays fully external and untouched until the model is locked, per your instruction.

## 5. LBNL vs. 100-Air-Purifiers domain-shift documentation

| | LBNL (primary) | 100 Air Purifiers (external test) |
|---|---|---|
| PM2.5 sensor | MetOne photometer (light-scattering, reference-adjacent) | Unspecified "built-in sensor" in commercial air purifier — make/model/method not disclosed, no calibration/validation reported |
| Units/resolution | µg/m³, continuous 1-min | µg/m³ (1 µg/m³ resolution), hourly averages only |
| Outdoor PM2.5 | Yes, paired per home | **None** — indoor-only |
| Other variables | CO2, formaldehyde, NO2, temp/RH, equipment operation | Formaldehyde, TVOC, temp/RH, purifier on/off + airflow — **no CO2** |
| Geography/context | 70 new-construction CA homes, windows-closed, mechanical ventilation | 100 homes, 18 Chinese provinces, 4 climate zones, real-world (non-controlled) conditions |
| Duration | ~1 week/home | 1 year/home |

This is a substantial domain shift: different country, different PM2.5 sensor class with no disclosed calibration, no outdoor PM2.5 at all (can't test infiltration/generalization on it, only pure indoor-forecast generalization), and hourly vs. 1-min resolution (its own held-out evaluation there would need a coarser horizon, e.g. 2–3 hours, not 30–60 min). Recommend reporting this as a transparent limitation of the external generalization test rather than tuning anything against it.

## Bottom line

All five pre-modeling checks confirm the original Step 1 plan is sound with the refinements above (35 µg/m³/15-min primary + per-home-relative sensitivity definition, 10-min-smoothed outdoor PM2.5 feature, ~10-home final holdout). Ready to proceed to Step 2 baselines once you sign off on the episode definition choice.
