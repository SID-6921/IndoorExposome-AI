# IndoorExposome AI — Round 2: External Validation, Calibration Correction, Sensitivity Analysis

## 1. 100-Air-Purifiers external validation (highest priority item) — headline result

**This is a genuinely positive, statistically well-supported cross-dataset generalization result — and a materially stronger evidentiary basis than the LBNL holdout, because the sample is 10x larger (100 homes, 10,712 qualifying episodes vs. 10 homes/10 episodes).**

A **separate, portable frozen model** (`FROZEN_CONFIG_PORTABLE.md` — distinct from the original `FROZEN_CONFIG.md`, do not conflate the two) was trained on LBNL using only features that exist in both datasets: indoor PM2.5 history (lags/rolling stats/rate-of-change/hours-since-threshold) plus current temperature and relative humidity. Since LBNL is 1-min and the external set is hourly, LBNL was first aggregated to hourly means so the two feature sets are constructed identically. The operating threshold (0.4121) and episode definition (35 µg/m³, adapted to ≥1 sustained hour at this resolution) were locked from LBNL alone before ever touching the 100-homes data — evaluated once, no tuning.

| Horizon | Event sensitivity | False alerts/home-day | Alert precision | Median lead | AUPRC | AUROC |
|---|---|---|---|---|---|---|
| **3h (primary)** | **56.5%** [50.0%, 60.5%] | **1.52** [1.28, 1.74] | 9.6% [7.1%, 11.9%] | 2.0h | 0.127 [0.091, 0.157] | **0.786** [0.752, 0.810] |
| 2h (secondary) | 49.9% [44.1%, 53.5%] | 2.03 [1.70, 2.35] | 6.7% [4.9%, 8.3%] | 2.0h | 0.087 [0.062, 0.108] | 0.784 [0.750, 0.808] |

(95% bootstrap CIs, home-level cluster resample, n=500.)

**At the primary 3-hour horizon, this clears an hourly-resolution analog of the operational benchmark** — ≥50% detection (56.5%, CI lower bound right at 50%), ≤2 false alerts/home-day (1.52, CI upper bound 1.74, comfortably under 2), with ~2 hours of real advance warning (2/3 of the nominal 3-hour horizon — a *better* fraction of the horizon achieved than the LBNL holdout managed, proportionally). Critically, **the AUROC confidence interval stays well above chance** (0.752–0.810) — unlike the LBNL holdout, where the small sample let the CI dip below 0.5. The much larger episode count here gives real statistical weight to this result.

**Caveats to keep in the narrative:**
- Alert precision remains low (9.6%) — same pattern as LBNL, this is a rate-bounded rather than precision-bounded system.
- **Group F (purifier on/off, airflow) was excluded from the trained model** — no LBNL equivalent existed to train against (LBNL homes ran continuous mechanical ventilation with no on/off variability). Including it would have required training directly on the 100-homes data, which would break the "pure external test" design. This is flagged in `FROZEN_CONFIG_PORTABLE.md`.
- The 35 µg/m³ threshold sits at ≈p95.7 of the 100-homes pooled distribution (vs. p97.5 in LBNL) — still a genuine tail event, but somewhat less extreme here, consistent with generally higher ambient PM2.5 across the 21 Chinese cities in this dataset vs. new-construction California homes.
- Domain-shift factors already documented in Step 1.5 remain: different PM2.5 sensor (undisclosed/uncalibrated vs. LBNL's MetOne photometer), hourly vs. 1-min native resolution, no CO2, no outdoor PM2.5 (so infiltration/protection modeling still can't be tested on this set).

## 2. Calibration correction pass

Before/after calibration slopes (ideal = 1.0), computed via proper out-of-fold Platt scaling and isotonic regression on the existing Step 2B dev-pool OOF probabilities (grouped by home, no leakage):

| Model | Before | After Platt | After isotonic |
|---|---|---|---|
| Logistic regression | 0.560 | **0.955** | 0.870 |
| Random Forest | 0.663 | **0.956** | 0.903 |
| XGBoost | 0.280 | **0.954** | 0.879 |

Platt scaling brings all three models to within ~0.05 of ideal calibration (slope ≈1.0), a substantial, concrete improvement. This lets the narrative say the miscalibration has already been corrected, not merely identified. As before, this is a secondary technical finding — it doesn't change the primary event-level GO/NO-GO metrics, which depend on ranking rather than literal probability values.

## 3. Per-home relative threshold sensitivity analysis (p90, 12 µg/m³ floor)

This is the one piece of real bad news in this round, and it should be reported plainly: **under the sensitivity episode definition, the model does not clear the same benchmark it cleared under the primary definition.**

| | Primary (35 µg/m³ fixed) | Sensitivity (per-home p90, 12µg floor) |
|---|---|---|
| Qualifying episodes | 130 | 245 |
| Homes with ≥1 episode | 35/59 (59%) | 47/58 (81%) |
| Event sensitivity @ ≤2 FA/day | 59.2% | **45.3%** (below ≥50% bar) |
| Median first-alert lead | 16.0 min | **12.0 min** (below 15–20 min bar) |
| Per-minute AUPRC / AUROC | 0.0437 / 0.755 | 0.0592 / 0.725 |

The sensitivity definition surfaces nearly twice as many episodes (245 vs. 130) and covers more homes (81% vs. 59%) — consistent with it correcting for inter-home baseline heterogeneity, as intended. But the *harder-to-detect* additional episodes it captures (many closer to each home's own baseline rather than an absolute extreme) pull both detection rate and lead time below the benchmark. This means the primary result is somewhat sensitive to which episode definition is used — worth stating directly rather than only reporting the more favorable primary number.

## 4. Why RF/XGBoost clear the benchmark at 60 min but not 45 min

See attached `horizon_explanation.md` — short version: the same 130 episodes exist regardless of horizon; widening the alert-matching window from 45 to 60 minutes mechanically (a) gives more chances for an already-fired alert to fall inside the window, and (b) makes alerts fired further in advance newly eligible to count. RF: 62.3%/14.0min (45min) → 70.0%/20.0min (60min). This should be read as "the same model, wider credit-matching window," not "a materially better model at 60 minutes."

## 5. COLLECTiEF NDA access outreach

Drafted, contact confirmed as the actual project leader (Dr. Mohammadreza Aghaei, mohammadreza.aghaei@ntnu.no) via the Zenodo record. Ready to send — see the message provided separately.

## 6. Light literature/market notes

Not yet started — lower priority per your email, flagging as still open. Can pick this up next if useful before submission.

## Recommendation

Item 1 (100-homes external validation) is a genuine strengthening of the aims page: it's a larger, statistically tighter, independently-sensored replication of the primary finding, with the caveats above traveling alongside it. Items 2–4 are ready to fold into the narrative as-is. Item 3 is the one result that should temper any "the benchmark is robustly met" framing — the primary definition clears it, the sensitivity definition doesn't, and that dependency-on-definition is itself useful, honest information for a GO/NO-GO judgment.
