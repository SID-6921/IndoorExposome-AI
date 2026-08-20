# IndoorExposome AI: Step 2 Preliminary Baseline Modeling Results

**Scope:** Group k-fold cross-validation (5-fold, grouped by home) on the 59-home CV pool only. The 10-home final LBNL holdout and the 100-Air-Purifiers external dataset were **not touched**: no feature selection, threshold selection, or hyperparameter tuning used them. This report is sent for your review before either is opened, per your instruction.

**Locked settings used:** episode = indoor PM2.5 ≥35 µg/m³ for ≥15 consecutive min; primary horizon 45 min (secondary: 30, 60 min); outdoor PM2.5 smoothed with a 10-min trailing rolling mean (LBNL-recommended, computed causally); all features use only information available at or before the prediction time.

## Headline answer to the core question

**Preliminary answer: not yet. The current preliminary models do not clearly and consistently outperform simple persistence, and the practical false-alarm rate is too high for a usable product signal.** This needs to be reported plainly rather than framed as a win, per your standing instruction.

- On **AUPRC** (the primary metric, given ~1% positive prevalence), the best ML models (Random Forest, XGBoost) beat persistence by roughly **2x** (e.g. at the primary 45-min horizon: persistence AUPRC=0.025 vs. RF=0.057, XGBoost=0.044), a real but modest relative gain over a low absolute baseline.
- On **AUROC**, the picture is much weaker: XGBoost actually scores **below** plain persistence at every feature group tested (e.g. 0.619–0.646 vs. persistence's 0.709–0.716). Only logistic regression (Model D) and RF (Model D) exceed persistence's AUROC, and only modestly (0.742 and 0.723 vs. 0.709–0.716).
- **False alarms remain high even at the best operating point.** At the primary horizon, RF/XGBoost still produce ~14 false alarms per home-day; persistence/threshold-rule produce ~59–61/day. The single best configuration found (RF, 30-min horizon) still produces **~6.6 false alarms per home-day** at its best-F1 threshold, far too high for a consumer alerting product as currently built.
- **Achieved lead time is much shorter than the nominal horizon.** At the 45-min horizon, true positives from RF average only **13.9 min** of real advance warning (median 11 min) before onset, not 45. XGBoost: mean 16.3 min. Logistic regression: mean 19.4 min (longest of the three, despite weaker AUPRC). In other words, the models function closer to a short-horizon "onset is imminent" detector than a 30–60-minute-ahead forecaster.

## Full ablation table: primary horizon (45 min)

| Model | Feature group | AUPRC | AUROC | Sensitivity | Specificity | Precision | F1 | False alarms/home-day |
|---|---|---|---|---|---|---|---|---|
| Persistence | — | 0.0250 | 0.709 | 0.197 | 0.955 | 0.045 | 0.073 | 60.7 |
| Threshold rule | — | 0.0254 | 0.716 | 0.197 | 0.956 | 0.046 | 0.074 | 59.1 |
| Logistic regression | A (indoor only) | 0.0259 | 0.641 | 0.206 | 0.955 | 0.046 | 0.076 | 60.7 |
| Logistic regression | B (+outdoor) | 0.0276 | 0.654 | 0.148 | 0.978 | 0.068 | 0.093 | 29.1 |
| Logistic regression | C (+environmental) | 0.0289 | 0.701 | 0.135 | 0.978 | 0.061 | 0.084 | 29.9 |
| Logistic regression | D (full) | 0.0415 | **0.742** | 0.240 | 0.970 | 0.078 | 0.118 | 40.7 |
| Random Forest | A | 0.0538 | 0.643 | 0.091 | 0.996 | 0.179 | 0.120 | 6.0 |
| Random Forest | B | 0.0504 | 0.657 | 0.109 | 0.991 | 0.115 | 0.112 | 12.0 |
| Random Forest | C | 0.0513 | 0.690 | 0.134 | 0.990 | 0.121 | 0.128 | 14.0 |
| Random Forest | D | **0.0573** | 0.723 | 0.136 | 0.989 | 0.121 | 0.128 | 14.2 |
| XGBoost | A | **0.0548** | 0.619 | 0.091 | 0.996 | 0.196 | 0.124 | 5.4 |
| XGBoost | B | 0.0472 | 0.643 | 0.082 | 0.996 | 0.167 | 0.110 | 5.9 |
| XGBoost | C | 0.0353 | 0.637 | 0.110 | 0.986 | 0.078 | 0.091 | 18.7 |
| XGBoost | D | 0.0442 | 0.646 | 0.108 | 0.990 | 0.102 | 0.105 | 13.6 |

Bolded = best in column. Note the internal inconsistency: **XGBoost's best AUPRC is at feature group A (indoor-only)**. Adding outdoor/environmental/activity features makes XGBoost *worse* on this metric, the opposite of what the ablation was designed to show. Random Forest and logistic regression both do best at full feature group D, but the size of that improvement is small for RF (0.0538→0.0573, +6%) and larger for logistic regression (0.0259→0.0415, +60%). The multimodal-value story holds for logistic regression but is not confirmed for the stronger tree-based models.

## Secondary horizons (feature group D only)

| Horizon | Model | AUPRC | AUROC | False alarms/home-day | Lead time: mean / median (min) |
|---|---|---|---|---|---|
| 30 min | Persistence | 0.019 | 0.738 | 61.5 | — |
| 30 min | Logistic regression | 0.033 | **0.770** | 29.1 | — |
| 30 min | Random Forest | **0.074** | 0.756 | 6.7 | 10.0 / 8.0 |
| 30 min | XGBoost | 0.056 | 0.672 | 7.8 | — |
| 45 min | Random Forest | 0.057 | 0.723 | 14.2 | 13.9 / 11.0 |
| 60 min | Persistence | 0.031 | 0.690 | 49.6 | — |
| 60 min | Random Forest | 0.064 | 0.689 | 15.2 | 18.3 / 13.0 |

The 30-min horizon gives the single best preliminary result found (RF AUPRC=0.074, best false-alarm rate of ~6.7/home-day), consistent with the general pattern that shorter horizons are easier to predict. This is worth flagging as a secondary-analysis finding: if the eventual product needs to trade horizon length for reliability, 30 min looks materially more tractable than 45–60 min in this preliminary pass.

## Feature importance (descriptive: Random Forest, feature group D, 45-min horizon, fit on the full 59-home CV pool; not a held-out performance claim)

Top drivers: **cooktop burner temperature** (rolling max over 30 min, and current value, the two single strongest features), followed by indoor relative humidity, CO2 (rolling mean and current), indoor temperature, then indoor PM2.5 rolling variability (std over 60/15 min) and smoothed outdoor PM2.5. This is a physically sensible result: cooking activity and ventilation/occupancy proxies (CO2, RH) carry real signal for upcoming PM elevation, which supports the multimodal premise conceptually even though it didn't translate into strong aggregate gains for RF/XGBoost in the ablation above.

## Calibration note

Brier scores for RF/XGBoost look good in absolute terms (0.02–0.09 vs. persistence's ~0.92–0.95), but this is expected under ~1% base rate: a model that always predicts a low probability will already have a low Brier score without necessarily discriminating well. Given the AUPRC/AUROC results above, low Brier score here should **not** be read as evidence of strong calibrated reliability; a proper reliability-diagram/decile analysis is recommended before any calibration claim is made in a grant narrative.

## Assessment against your 5 GO/NO-GO criteria

1. **Added predictive value over persistence/conventional baselines:** Mixed. ~2x AUPRC gain for RF/XGBoost, but AUROC gain is inconsistent (XGBoost underperforms persistence on AUROC at every feature group).
2. **Generalizes to unseen buildings:** The results above already reflect home-held-out generalization (group k-fold), so this is the generalization performance, and it's weak, not a separate hurdle still to clear.
3. **Multimodal value beyond indoor history alone:** Supported for logistic regression only; not confirmed for RF/XGBoost in this preliminary pass.
4. **Meaningful advance warning:** Weak. Real achieved lead times (11–19 min average) fall well short of the 30–60 min horizon the product concept describes.
5. **Reliability/calibration flags unreliable predictions:** Not yet properly evaluated. Brier scores alone are misleading here (see calibration note); a real reliability analysis is a prerequisite before drawing conclusions.

## Recommendation

This is a not-yet-positive result and should be reported to you as such rather than reframed. Before deciding whether to open the sealed 10-home holdout, worth considering: (a) whether the false-alarm rate and short lead time make the current formulation viable for the CleanStay AI product framing regardless of held-out performance, and (b) whether additional preliminary iteration (e.g. probability calibration, a proper reliability diagram, or revisiting the 30-min horizon as primary given its stronger showing) is worth doing on the CV pool *before* opening the holdout, since re-opening the holdout after changes would break the "locked before evaluation" principle you set.

Full code and per-fold outputs available on request; not yet pushed to the repo pending your read of these results.
