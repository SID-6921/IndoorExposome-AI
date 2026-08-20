# IndoorExposome AI - Step 2C: Frozen-Model Holdout Evaluation (10-Home LBNL Test Set)

**This is a single, one-shot evaluation.** The model, feature set, preprocessing, operating threshold (0.8712), and 30-min alert cooldown were frozen and committed to the repo (`FROZEN_CONFIG.md`) *before* the 10 holdout homes were loaded or touched in any way. Nothing below was changed after seeing these results: no re-thresholding, no retraining, no feature changes, no model reselection. This report reflects the holdout run exactly as it came out.

## Headline

**The point estimate clears your operational benchmark, but with only 10 episodes across 10 homes, the uncertainty is large enough that this should be read as encouraging, not confirmatory.**

| Metric | Holdout result | 95% bootstrap CI (home-level cluster resample, n=2000) |
|---|---|---|
| Number of holdout homes | 10 | — |
| Number of qualifying PM2.5 episodes | 10 | — |
| Event sensitivity | **60.0%** | **[6.3%, 93.3%]** |
| Distinct false alerts/home-day | **0.97** | [0.42, 1.74] |
| Alert precision | 11.1% | [0.0%, 26.5%] |
| Median first-alert lead time | **18.0 min** | [12.0, 24.0] |
| Mean first-alert lead time | 19.0 min | — |
| % episodes warned ≥10 min early | 66.7% | — |
| % episodes warned ≥20 min early | 50.0% | — |
| % episodes warned ≥30 min early | 33.3% | — |
| AUPRC (per-minute) | 0.0329 | [0.0029, 0.0855] |
| AUROC (per-minute) | **0.620** | **[0.294, 0.844]** |

**On the point estimate, all three benchmark conditions are met:** 60.0% detection (≥50%), 0.97 false alerts/home-day (≤2), and 18.0 min median lead (≥15–20 min). That matches the development-pool result (59.2% / 1.91 / 16.0 min), a reassuring sign that the frozen model isn't behaving wildly differently on unseen homes.

**But the confidence intervals need to be read carefully and reported honestly:**
- With only **10 qualifying episodes total** in the holdout (6 of the 10 homes had zero episodes at all), the event-sensitivity estimate is extremely uncertain: the 95% CI runs from 6.3% to 93.3%. A single episode detected or missed differently shifts the point estimate by 10 percentage points. This does not confirm ≥50% detection; it is one plausible draw consistent with a wide range of true detection rates.
- **The AUROC confidence interval's lower bound (0.294) is below chance (0.5).** This means the holdout data cannot rule out the possibility that the model discriminates no better than random on unseen homes. The point estimate (0.620) is encouraging, but the sample is too small to be confident in it.
- False-alerts/home-day and median lead time have comparatively tighter, more reassuring intervals (both stay within or near the target range across most of the distribution).

## What this does and doesn't tell us

This is a favorable single holdout draw against a small, honestly-reported sample. It does not yet constitute strong statistical confirmation of generalization. The wide intervals are a direct, correct consequence of only 10 qualifying episodes existing in the 10 sealed homes, not a flaw in the analysis. A larger holdout (or pooling in the 100-Air-Purifiers data once its domain differences are accounted for) would be needed to narrow these intervals before this could support a strong generalization claim in a grant narrative.

## Reminder of what was frozen (see `FROZEN_CONFIG.md`, committed prior to this run)

Feature set: Full+ (A indoor history + B outdoor-smoothed + C environmental + D cooktop-activity + E hypothesis-driven slope/interaction features), 35 features total. Preprocessing: median imputation + standard scaling, fit on the 59-home development pool only. Model: logistic regression, `class_weight="balanced"`, trained on all 59 development-pool homes. Operating threshold: 0.8711552648952829 (the ≤2-FA/home-day point from the development pool). Alert cooldown: 30 minutes. Primary horizon: 45 minutes. Episode definition: indoor PM2.5 ≥35 µg/m³ for ≥15 consecutive minutes. None of this was altered after seeing the numbers above.

## 100-Air-Purifiers dataset

Remains completely sealed, as instructed. Not opened or touched in this step.

## Recommendation

The point estimate is a positive result and consistent with the development-pool finding. But given the small episode count, I'd recommend treating this as "encouraging preliminary evidence" rather than "confirmed generalization" in any grant narrative. The honest statistical statement is that the true detection rate on unseen homes could plausibly be anywhere from ~6% to ~93% given this sample, and AUROC's CI can't rule out chance-level discrimination. Whether that's sufficient to proceed, and how to word it for the STTR narrative, is your call. Happy to discuss options (e.g., a larger validation sample, or explicitly reporting the CIs alongside the point estimate) before finalizing that language.
