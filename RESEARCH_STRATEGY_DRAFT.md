# Research Strategy / Approach — Draft (Aims 1 & 2)

**Status: structural draft to seed the full Research Strategy section — not final language, built to be expanded/edited.** Grounded entirely in the preliminary data already in this repo; every claim below cites the specific document it comes from.

---

## Aim 1: Characterize and Resolve Regime-Dependent Detection Performance in Indoor PM2.5 Episode Forecasting

### Rationale and preliminary data

Preliminary work established that a single, fixed episode-detection threshold does not perform uniformly across buildings with different baseline air-quality regimes — and, critically, that this regime-sensitivity itself is a real, replicating phenomenon, even though its *specific direction* is not.

- In the LBNL development cohort, a 6-home "high-baseline" subset (homes whose own 90th-percentile PM2.5 already exceeds the 35 µg/m³ episode threshold) showed markedly *lower* event-detection performance (13.6%) than the 52-home "low-baseline" majority (45.0%) (`REGIME_STRATIFIED_CHECK.md`).
- Testing the *same* frozen model and threshold on an independent, 100-home, cross-national external cohort replicated the low-baseline result almost exactly (45.6%) but **inverted** the high-baseline result (68.0% detection, at a false-alarm rate 60% above the operational target) (`REGIME_REPLICATION_100HOME.md`).
- Separately, testing whether the underlying 35 µg/m³ absolute threshold, a per-home relative threshold, or a union of both best resolves this was itself inconclusive with adequate statistical rigor: a home-level cluster bootstrap showed the fixed and relative definitions are not statistically distinguishable from each other (paired 95% CI [−2.1, +27.5] pp), while a naive union of the two is confidently *worse* than either alone (`DEFINITION_BOOTSTRAP_COMPARISON.md`, `HYBRID_DEFINITION_FIRST_PASS.md`).

Together, these findings support a specific, falsifiable claim: **a single fixed-threshold, single-model approach to episode detection is not robust to the between-building heterogeneity that any real multi-site deployment will encounter — but the data currently available are not sufficient to characterize *why*, or to select among candidate remedies with confidence.** That characterization and remedy-selection is the substance of Aim 1.

### Specific approach

**Sub-aim 1a — Acquire and characterize additional high-baseline-regime buildings.** The central limitation across all definition/regime analyses to date is sample size in the high-baseline regime specifically (6 buildings in the development cohort, 13 in the external cohort) — too few to resolve competing hypotheses about *why* detection performance and false-alarm behavior diverge by regime. This sub-aim targets recruitment/acquisition of additional high-baseline buildings (candidate sources: remaining unexploited LBNL homes not yet regime-classified in this analysis, negotiated access to the COLLECTiEF cohort, and/or new data collection in partnership with the CleanStay AI pilot sites) sufficient to power a properly-designed regime comparison.

**Sub-aim 1b — Formally evaluate candidate regime-aware architectures.** With adequate per-regime sample size from 1a, systematically test the four candidate directions identified in preliminary technical scoping (`TECHNICAL_APPROACH_NOTE.md`), in order of implementation cost:
1. Baseline-level-as-input-feature (single pooled model, continuous regime signal)
2. Severity/regime-weighted training loss
3. Regime-conditioned mixture-of-experts (explicit regime classification + specialized sub-models)
4. Per-deployment adaptation layer (extends Aim 2's per-site calibration concept to detection thresholds)

Each will be evaluated under the same event-level, home-held-out, bootstrap-CI methodology already established and validated across three preliminary rounds, so that this aim's outputs are directly comparable to the preliminary numbers already reported.

**Sub-aim 1c — Establish a single, pre-registered episode-detection protocol.** Using the expanded dataset from 1a and the architecture comparison from 1b, lock a final episode definition and detection protocol robust to the between-building heterogeneity characterized above — replacing the current "fixed vs. relative vs. hybrid, unresolved" state with a defensible, validated final answer, pre-registered before any confirmatory evaluation on a final held-out cohort.

### Milestones
- M1: Acquired/curated high-baseline-regime dataset with adequate power (target: ≥25-30 high-baseline buildings, informed by an a priori power calculation)
- M2: Comparative evaluation of the 4 candidate architectures, event-level metrics + bootstrap CIs, reported transparently regardless of outcome
- M3: Locked, pre-registered unified episode-detection protocol
- M4: Confirmatory evaluation on a final untouched cohort (methodology precedent: `STEP2C_HOLDOUT_RESULTS.md`)

---

## Aim 2: Develop and Validate an Operational Per-Deployment Calibration Protocol

### Rationale and preliminary data

A probability calibrator (Platt scaling) fit on the development cohort does **not** transfer cleanly to an independent external cohort — it over-corrects (calibration slope 1.165 vs. ideal 1.0) rather than generalizing, while refitting locally on the target cohort restores near-ideal calibration (slope 1.002) (`CALIBRATOR_TRANSFER_TEST.md`). This means model *discrimination* (ranking ability, reflected in AUROC/AUPRC) may transfer reasonably across deployments, but model *calibration* (literal probability/risk-percentage claims) does not — a critical distinction for any product that reports numeric risk to end users. A first-pass simulation of how much per-property data is needed before local recalibration stabilizes was inconclusive (`RECALIBRATION_DATA_REQUIREMENT.md`) but suggestive of a 60–180-day / ~30-130-positive-window range, with a clear, identified methodological fix (fixed evaluation window) needed to determine this rigorously.

### Specific approach

**Sub-aim 2a — Design and validate the deployment break-in protocol.** New deployments will operate initially on the shared model's raw discriminative ranking (threshold-based alerting) rather than literal calibrated risk percentages, since ranking-based alerting does not require calibration to already be correct. Validate that this break-in mode provides acceptable alert-burden characteristics (false-alarms/property-day) using the same event-level methodology established in Aims/preliminary work.

**Sub-aim 2b — Rigorously determine the data-sufficiency threshold for local recalibration.** Repeat the recalibration-data-requirement simulation with the identified methodological correction (fixed-size, held-out evaluation window per training-budget step, rather than "all remaining data," which confounds the comparison as the training window grows) across a larger and more diverse set of properties (both cohorts, plus Aim 1's expanded high-baseline dataset). Deliverable: a validated, defensible data-sufficiency gate — expressed in accumulated positive-alert-windows rather than calendar time, since preliminary work suggests episode *count*, not elapsed time, is the operative limiting factor — for switching a property from break-in/ranking mode to calibrated/percentage mode.

**Sub-aim 2c — Build and validate ongoing calibration-drift monitoring.** Real deployments will experience seasonal ambient air-quality shifts and changing occupant behavior. Design and validate a monitoring mechanism (e.g., a rolling-window calibration-slope check) that flags when a property's calibration has drifted enough to warrant a refresh, and validate the refresh procedure itself using the same held-out methodology.

### Milestones
- M1: Validated break-in-mode alert-burden characteristics across both existing cohorts
- M2: Rigorous (fixed-window) recalibration data-sufficiency threshold, reported as a specific, defensible number
- M3: Validated drift-detection and refresh mechanism
- M4: End-to-end operational protocol specification, ready for integration into the CleanStay AI product deployment pipeline
