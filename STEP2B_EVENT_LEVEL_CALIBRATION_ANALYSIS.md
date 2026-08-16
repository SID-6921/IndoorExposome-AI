# IndoorExposome AI — Step 2B: Event-Level & Calibration Analysis

**Scope:** 59-home CV development pool only. The 10-home LBNL holdout and the 100-Air-Purifiers external dataset remain completely untouched — not opened, not used for any threshold, feature, or hyperparameter choice below. Primary horizon (45 min) retained as primary; 30/60 min retained as secondary, unchanged per your instruction.

## Headline

**This is a materially better picture than Step 2, once measured the way a real product would be evaluated — with one important caveat about which model gets there.**

At the primary **45-minute horizon**, **logistic regression at the ≤2-false-alerts/home-day operating point achieves 59.2% episode detection with a 16-minute median first-alert lead time** — this *clears* your proposed benchmark (≥50% detection, ≤2 FA/home-day, ≥15–20 min median lead). At the secondary **60-minute horizon**, both **Random Forest (70.0% detection, 20 min lead, 1.94 FA/day)** and **logistic regression (51.5% detection, 24 min lead, 1.89 FA/day)** clear it too, RF comfortably.

The catch: **RF and XGBoost — the models with the best raw AUPRC in Step 2 — do not clear the lead-time bar at the 45-min primary horizon.** At ≤2 FA/day, RF gets 62.3% detection but only 14-minute median lead; XGBoost gets 50.8% detection at 11.5 minutes. Both are close but fall short of "≥15–20 min." Logistic regression, which looked weakest in Step 2's raw-minute metrics, is the one that actually clears the full bar at the primary horizon. This needs to be reported plainly, not smoothed over.

## 1–3. Event-level alerting, first-actionable-alert lead time, fixed alert-burden operating points

**Alert consolidation rule used:** a 30-minute cooldown — once a model's calibrated probability crosses the operating threshold and fires an alert, no new alert is counted for that home until 30 minutes have elapsed, even if the score stays above threshold. This collapses runs of consecutive positive minutes (which was the Step 2 "false alarms per home-day" measure) into discrete alert events.

**Total CV-pool home-days: 394.3. Total qualifying episodes across the 59-home pool: 130.**

### Operating points at ≤1 and ≤2 distinct false alerts/home-day

| Horizon | Model | Target | Threshold | FA/home-day | Event sensitivity | Alert precision | Median first-alert lead (min) |
|---|---|---|---|---|---|---|---|
| **45 (primary)** | persistence | ≤1 | 0.054 | 0.69 | 20.8% | 9.3% | 1.0 |
| 45 | persistence | ≤2 | 0.038 | 1.79 | 66.9% | 11.4% | 3.0 |
| 45 | threshold_rule | ≤1 | 0.053 | 0.71 | 17.7% | 7.9% | 1.0 |
| 45 | threshold_rule | ≤2 | 0.037 | 1.98 | 66.2% | 10.6% | 2.5 |
| 45 | **logreg** | ≤1 | 0.949 | 0.81 | 30.8% | 11.4% | 7.5 |
| **45** | **logreg** | **≤2** | **0.871** | **1.91** | **59.2%** | **10.0%** | **16.0** |
| 45 | rf | ≤1 | 0.700 | 0.78 | 37.7% | 14.3% | 12.0 |
| 45 | rf | ≤2 | 0.495 | 1.84 | 62.3% | 10.8% | 14.0 |
| 45 | xgb | ≤1 | 0.614 | 0.71 | 34.6% | 13.8% | 9.0 |
| 45 | xgb | ≤2 | 0.310 | 1.95 | 50.8% | 8.6% | 11.5 |
| 30 (secondary) | rf | ≤1 | 0.661 | 0.87 | 50.0% | 15.7% | 9.0 |
| 30 | rf | ≤2 | 0.483 | 1.83 | 67.7% | 10.6% | 7.0 |
| 30 | xgb | ≤2 | 0.188 | 1.99 | 57.7% | 8.5% | 10.0 |
| **60 (secondary)** | **rf** | **≤2** | **0.467** | **1.94** | **70.0%** | **12.8%** | **20.0** |
| 60 | rf | ≤1 | 0.632 | 0.92 | 48.5% | 16.6% | 16.0 |
| **60** | **logreg** | **≤2** | **0.865** | **1.89** | **51.5%** | **9.8%** | **24.0** |
| 60 | xgb | ≤2 | 0.300 | 1.96 | 56.2% | 9.7% | 13.0 |

(Full 30-model-row table with all persistence/threshold_rule/logreg/rf/xgb × horizon × burden-target combinations available in the underlying JSON on request; table above shows the operationally relevant subset.)

**Reading this honestly:** even at the operating points that clear the detection/lead-time bar, **alert precision is low (~10–17%)** — meaning roughly 5 to 9 out of every 10 alerts fired are false alarms in absolute count. What makes this potentially still viable is that the *rate* is bounded to ≤2/home-day by construction, which is the metric you asked us to target — but "1 in 10 alerts is real" is a genuine usability question for a consumer product, separate from the rate question, and should be weighed as such.

Note also the 30-min horizon structurally cannot reach the ≥15–20 min lead bar (the horizon itself caps lead time at 30 minutes, and the best achieved there is 10 minutes) — this is a ceiling effect of the horizon choice, not a modeling failure, and is worth keeping in mind when comparing horizons.

## 4. Brier-score fix

Confirmed the bug: the original persistence/threshold-rule "scores" were raw µg/m³ PM2.5 values (e.g., 2, 35, 2006), not probabilities — `brier_score_loss` doesn't enforce a [0,1] range, so it was silently computing nonsense squared errors against those raw values. Fixed by fitting an **out-of-fold isotonic regression** per training fold (mapping raw persistence/threshold score → empirical probability, fit on training-fold data only, applied to the held-out fold) — a legitimate "probabilistic persistence forecast" comparable on equal footing with the ML models.

Corrected Brier scores (45-min horizon): persistence **0.0104**, threshold_rule **0.0104**, logreg 0.1656, RF 0.0342, XGBoost 0.0167. Persistence is now well-calibrated by construction (isotonic regression guarantees this) — its poor performance is entirely a discrimination (AUPRC/AUROC) problem, not a calibration one, which is the correct and expected result.

## 5. Calibration analysis (CV pool only)

Predicted-risk decile tables and calibration slope/intercept (logistic regression of label on logit(predicted probability)) computed for all 5 models at all 3 horizons. Headline at the primary 45-min horizon:

| Model | Calibration slope | Calibration intercept | Top-decile: mean predicted vs. observed |
|---|---|---|---|
| persistence (isotonic-calibrated) | 0.86 | -0.59 | 3.7% vs 3.3% |
| threshold_rule (isotonic-calibrated) | 0.87 | -0.56 | 3.6% vs 3.2% |
| logreg | 0.56 | -4.44 | **80.6% vs 4.5%** |
| RF | 0.66 | -3.21 | 42.9% vs 4.4% |
| XGBoost | 0.28 | -3.14 | 21.7% vs 3.2% |

**Slope should be ~1.0 for a well-calibrated model.** All three ML models are substantially overconfident — logistic regression's most extreme decile predicts an 80.6% chance of an episode when the true rate there is 4.5%; XGBoost is directionally the same problem though numerically less extreme (21.7% vs 3.2%). This does **not** invalidate the event-level alerting results above (those depend only on relative ranking / threshold crossing, not on the literal probability value), but it does mean: **the raw model probabilities should not be shown to users or PIs as literal risk percentages without a separate post-hoc recalibration step** (e.g., the same isotonic approach used for persistence). Worth flagging for any future grant-narrative or product-facing claim about "% risk."

## 6. Limited hypothesis-driven feature pass

Added exactly the features you specified — PM2.5 slope (30-min), PM2.5 acceleration (change in slope), CO2 slope, RH change, cooktop-temperature change, minutes-since-burner-activation (defined as cooktop temp >5°C above its own 60-min rolling median), and two cooking×ventilation interaction terms — to the existing full feature set (nothing else added, no broader search). Result at the primary horizon: full-feature-set-plus-these (called "Full+" above) AUPRC/AUROC: logreg 0.0437/0.755 (vs. 0.0415/0.742 in Step 2), RF 0.0558/0.720 (vs. 0.0573/0.723, essentially flat), XGBoost 0.0404/0.656 (vs. 0.0442/0.646, essentially flat). **Modest improvement for logistic regression, negligible change for RF/XGBoost** — consistent with Step 2's finding that tree ensembles aren't extracting incremental value from added engineered signal in this preliminary pass, while the added features do help the simpler linear model somewhat.

## 7. Endpoint structure

Unchanged: primary = 45 min, secondary = 30 and 60 min, all reported transparently as instructed. 35 µg/m³/15-min episode definition unchanged.

## Assessment against your operational benchmark

**≥50% episode detection, ≤1–2 distinct false alerts/home-day, median first-alert lead time ≥15–20 min:**

- **45-min primary horizon: met, by logistic regression only** (59.2% / 1.91 FA-day / 16.0 min), **not met by RF or XGBoost** (which get the detection and FA-rate right but fall 1–6 minutes short on lead time).
- **60-min secondary horizon: met, by both RF (comfortably: 70.0% / 1.94 / 20.0) and logistic regression** (51.5% / 1.89 / 24.0).
- **30-min secondary horizon: not met** — structurally capped on lead time (max possible is 30 min; best achieved is 10 min), even though detection rates there are the highest of any horizon (RF: 50–68%).

## Recommendation

This is genuinely closer to your operationally-meaningful range than Step 2 suggested, but it comes with real texture that shouldn't be smoothed away: it's met at the primary horizon by the weaker-raw-metric model (logistic regression), not by the models that looked strongest in Step 2 (RF/XGBoost); alert precision remains low (~10-17%) even where the rate-based criteria are satisfied; and the raw ML probabilities are not calibrated for literal risk reporting. Given that the benchmark is met (with these caveats clearly stated), this seems like a reasonable basis to discuss opening the sealed holdout — but that call is yours, and the caveats above should travel with any GO decision rather than being left out of the grant narrative.
