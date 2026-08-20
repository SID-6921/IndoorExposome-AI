# Calibrator transfer test: LBNL → 100-home cohort

**One run, as requested: early signal, not fully finished analysis.**

The Platt-scaling calibrator (a logistic regression mapping raw model logit to calibrated probability) was fit on the portable model's own LBNL dev-pool out-of-fold predictions, then applied, unmodified, to the portable model's raw predictions on the 100-home external cohort.

| | Calibration slope (ideal = 1.0) | Calibration intercept (ideal = 0.0) |
|---|---|---|
| Raw, uncalibrated 100-home output | 0.728 | −3.028 |
| **LBNL-fit calibrator applied as-is** | **1.165** | **0.908** |
| Calibrator refit directly on 100-home data (in-sample, best-case reference) | 1.002 | 0.005 |

**Answer: it does not transfer well as-is, it needs refitting.** The raw model is under-confident on the external cohort (slope 0.728, same direction as the original LBNL miscalibration). Applying the LBNL-fit correction overshoots past ideal (slope 1.165, intercept +0.908). The correction was tuned to LBNL's specific miscalibration pattern, and that pattern differs enough on the 100-home cohort (different sensor, country, climate, resolution) that the fix overcorrects rather than landing cleanly. Refitting a calibrator directly on the target cohort's own data, by contrast, restores near-ideal calibration (slope 1.002).

**Practical read:** calibration appears to be dataset/deployment-specific rather than something that travels with the model. Any real deployment to a new environment (a new hotel chain, a new sensor vendor) would likely need its own recalibration pass rather than inheriting one fit elsewhere. This is consistent with treating this as an early signal: one run, not an exhaustive study of *why* the correction overshoots or whether a different calibration method transfers better.
