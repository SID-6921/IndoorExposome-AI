# IndoorExposome AI

Preliminary computational feasibility work for a proposed NIEHS STTR Phase I concept: **Nexara IndoorExposome AI™**, with **CleanStay AI™** as the initial hospitality application (PI: Narasaiah Kolliputi).

Central feasibility question: can multimodal AI predict sustained indoor PM2.5 exposure episodes 30–60 minutes in advance in previously unseen buildings, better than conventional forecasting approaches?

Status: **Step 1 complete** (dataset & endpoint feasibility), **pre-modeling confirmation checks complete** (see [STEP1_5_PRE_MODELING_CONFIRMATION.md](STEP1_5_PRE_MODELING_CONFIRMATION.md)), **Step 2 preliminary baseline modeling complete** (see [STEP2_PRELIMINARY_BASELINE_RESULTS.md](STEP2_PRELIMINARY_BASELINE_RESULTS.md)), **Step 2B event-level/calibration analysis complete** (see [STEP2B_EVENT_LEVEL_CALIBRATION_ANALYSIS.md](STEP2B_EVENT_LEVEL_CALIBRATION_ANALYSIS.md)).

**Step 2 headline (raw per-minute classification metrics):** current models do not clearly/consistently outperform a simple persistence baseline on AUROC, false-alarm rates are high per-minute, and average lead time across all flagged windows falls short of the 30–60 min horizon. Reported as-is per PI instruction not to force a positive result.

**Step 2B headline (event-level alerting with a 30-min cooldown, fixed alert-burden operating points, first-actionable-alert lead time — all on the 59-home CV pool only, holdout and 100-Air-Purifiers still sealed):** the picture improves once measured this way. At the **primary 45-min horizon**, logistic regression at ≤2 false alerts/home-day achieves **59.2% episode detection with a 16-minute median first-alert lead time** — clearing the PI's proposed operational benchmark (≥50% detection, ≤2 FA/day, ≥15–20 min lead). RF/XGBoost (the stronger models in Step 2) get the detection/FA-rate right but fall 1–6 minutes short of the lead-time bar at this horizon. At the **60-min secondary horizon**, both RF (70.0%/20 min) and logistic regression (51.5%/24 min) clear the benchmark comfortably. Calibration analysis also found the raw ML probabilities are substantially overconfident (calibration slopes 0.28–0.66, should be ~1.0) — fine for ranking/alerting, not for literal risk-percentage reporting without further recalibration. Awaiting PI review before opening the final holdout.

## Step 1: Dataset & Endpoint Feasibility Report

**Bottom line:** Feasible, but no single dataset satisfies every criterion at once. Recommend proceeding to Step 2 with a two-dataset strategy and a pre-registered episode definition below, pending sign-off.

### Recommended primary dataset

**LBNL California Homes Ventilation & IAQ Dataset** (Dryad, DOI [10.7941/D1ZS7X](https://datadryad.org/stash/dataset/doi:10.7941/D1ZS7X)) — 70 detached CA homes (built 2011–2017), each monitored ~1 week, with simultaneous indoor + outdoor PM2.5, indoor CO2, formaldehyde, NO2/NOx, plus mechanical-ventilation airflow/leakage diagnostics. Open download, no access gate. Only source found that pairs indoor+outdoor PM2.5 with ventilation context across a large enough building count (n=70) for genuine building-held-out cross-validation.

- **Open item:** sampling frequency not yet confirmed from the dataset files — needs direct inspection to confirm it supports a 30–60 min horizon (need ≥3x the horizon in lead samples). If hourly, horizon shifts to 60–120 min.
- **Caveat:** windows-closed protocol, 1 week/home, CA-only — limits generalization claims about natural ventilation and other climates.

### Secondary / robustness dataset

**"100 Air Purifiers" dataset** (China, Figshare [10.6084/m9.figshare.24278101](https://doi.org/10.6084/m9.figshare.24278101)) — 100 independent homes, 18 provinces, 4 climate zones, 1 year, hourly PM2.5/formaldehyde/TVOC + purifier on/off state. Fully open. Largest held-out-building sample size found (n=100) and most geographic diversity — best used as an out-of-distribution generalization test. Hourly resolution too coarse to serve as the primary 30–60 min forecasting corpus alone.

### Stretch / validation dataset

**COLLECTiEF** (Zenodo [10.5281/zenodo.16810954](https://doi.org/10.5281/zenodo.16810954)) — 14 buildings / 56 zones across Norway, Italy, France, Cyprus; 1-min CO2 + TVOC + PM1/2.5/4/10 + temp/RH; 2 years of data. Richest multi-building continuous PM2.5 dataset found, but **NDA-gated** (non-commercial research use, requires contacting the NTNU project coordinator) and has no outdoor PM2.5 or contextual (HVAC/occupancy) variables. Worth pursuing in parallel; should not be assumed available for a Step 1 commitment.

### Ruled out (verified, do not reconsider without new information)

| Dataset | Reason excluded |
|---|---|
| CU-BEMS | No PM2.5/CO2/VOC at all — energy + temp/RH/light only |
| ASHRAE Global Thermal Comfort DB II | No pollutants; point-in-time comfort votes, not continuous streams |
| OFFICAIR / SINPHONIE | Discrete campaign snapshots, not continuous time series — unusable for forecasting |
| HOMEChem / Harvard Healthy Buildings | Single building, or no public dataset release found |

### Proposed exposure-episode definition (for PI review, to be locked before modeling)

- **Threshold:** indoor PM2.5 ≥ 35 µg/m³ (EPA 24-hr NAAQS level, standard acute-elevation proxy in episodic-exposure literature)
- **Minimum duration:** ≥15 consecutive minutes above threshold (≥2 consecutive readings if coarser sampling forces it)
- **Prediction horizon:** 30–60 min, exact value contingent on confirming the primary dataset's sampling interval
- **Task framing:** binary classification — "episode onset within horizon window" vs. persistence/no-onset

### Validation strategy

Group k-fold by building/site, never by timestamp — e.g. leave-15-buildings-out folds on the LBNL set, with the 100-homes dataset held out entirely as an external test set never touched during model selection or tuning.

### Secondary endpoints

- **Indoor/outdoor infiltration modeling** — supported by the LBNL dataset (has both signals).
- **CO2-based ventilation-event prediction** — no dataset combines multi-building scale + CO2 + explicit HVAC/window state. Best partial fits: HPDmobile (6 homes, occupancy but no HVAC/window state) or Aalborg University's Danish office dataset (rich HVAC/window state, single building only). Treated as exploratory/secondary per PI direction.

### Exploratory task: Dr. Irfan Rahman's published exposure-toxicology data

Reviewed 9 papers spanning oxidative stress, mitochondrial dysfunction, cytokines, epithelial injury, and cytotoxicity. Only one has a confirmed open data repository:

- **Herbert et al. 2023** (AJP-Lung, PMC10026991) — Figshare deposit [10.6084/m9.figshare.20323689.v2](https://doi.org/10.6084/m9.figshare.20323689.v2), multi-concentration dose-response across cytotoxicity, oxidative stress, and mitochondrial bioenergetics (Seahorse OCR/ECAR).
- **Kaur/Rahman 2025** (eLife, PMC12854674) — public GEO deposit **GSE263903**, single-cell transcriptomics.

The remaining 7 papers surveyed are figure-embedded or "available upon request" only — real, relevant endpoint panels for the STTR narrative, but not raw downloadable data today.

### Recommendation

Data support a genuine feasibility test. Proposed lock-in: LBNL dataset as primary corpus (pending sampling-frequency confirmation), 100-homes dataset as external generalization test, 35 µg/m³ / 15-min / 30–60-min episode definition, building-held-out cross-validation. Proceed to Step 2 baselines once confirmed by PI.
