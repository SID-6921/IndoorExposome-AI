# Research Strategy / Approach — Draft (Aims 1 & 2)

Status: structural draft to seed the full Research Strategy section, not final language. Every claim below cites the specific preliminary-data document it comes from.

---

## Aim 1: Characterize and resolve regime-dependent detection performance in indoor PM2.5 episode forecasting

### Rationale and preliminary data

Preliminary work found that a single fixed episode-detection threshold doesn't perform uniformly across buildings with different baseline air-quality regimes. More importantly, that regime-sensitivity is itself a real, repeatable phenomenon, even though its specific direction isn't.

In the LBNL development cohort, a 6-home "high-baseline" subset (homes whose own 90th-percentile PM2.5 already exceeds the 35 µg/m³ episode threshold) showed markedly lower event-detection performance (13.6%) than the 52-home "low-baseline" majority (45.0%) (`REGIME_STRATIFIED_CHECK.md`). Testing the same frozen model and threshold on an independent, 100-home, cross-national external cohort replicated the low-baseline result almost exactly (45.6%) but inverted the high-baseline one: 68.0% detection, at a false-alarm rate 60% above the operational target (`REGIME_REPLICATION_100HOME.md`).

Separately, we tested whether the fixed 35 µg/m³ threshold, a per-home relative threshold, or a union of both resolves this best, and that comparison was itself inconclusive with proper statistical rigor. A home-level cluster bootstrap showed the fixed and relative definitions aren't statistically distinguishable from each other (paired 95% CI −2.1 to +27.5 percentage points), while a naive union of the two is confidently worse than either alone (`DEFINITION_BOOTSTRAP_COMPARISON.md`, `HYBRID_DEFINITION_FIRST_PASS.md`).

Together, these findings support a specific, falsifiable claim: a single fixed-threshold, single-model approach to episode detection isn't robust to the between-building heterogeneity that any real multi-site deployment will encounter, and the data available so far aren't enough to say why, or to choose among candidate fixes with confidence. Characterizing that and choosing a remedy is the substance of Aim 1.

### Specific approach

**Acquire and characterize more high-baseline-regime buildings.** The limiting factor across every definition and regime analysis so far is sample size in the high-baseline regime specifically: 6 buildings in the development cohort, 13 in the external cohort. That's too few to tell apart competing explanations for why detection and false-alarm behavior diverge by regime. This sub-aim targets recruiting or acquiring enough additional high-baseline buildings (candidates: LBNL homes not yet regime-classified in this analysis, negotiated access to the COLLECTiEF cohort, new data collected with CleanStay AI pilot sites) to properly power a regime comparison.

**Formally evaluate candidate regime-aware architectures.** With adequate per-regime sample size, systematically test the four candidates identified in preliminary scoping (`TECHNICAL_APPROACH_NOTE.md`), roughly in order of implementation cost: baseline level as an input feature to a single pooled model; a severity- or regime-weighted training loss; a regime-conditioned mixture of experts (explicit regime classification routed to specialized sub-models); and a per-deployment adaptation layer that extends Aim 2's calibration concept to detection thresholds. Each gets evaluated under the same event-level, home-held-out, bootstrap-CI methodology already used in preliminary work, so the results are directly comparable to what's already reported.

**Lock a single, pre-registered episode-detection protocol.** Using the expanded dataset and the architecture comparison above, settle on a final episode definition and detection protocol that holds up across the heterogeneity we've characterized, replacing the current unresolved fixed-versus-relative-versus-hybrid state with a validated answer, pre-registered before any confirmatory evaluation on a final held-out cohort.

### Milestones

- Curated high-baseline-regime dataset with a target sample size to be finalized via power analysis, informed by the effect sizes observed in preliminary regime comparisons (see `POWER_CALCULATION_NOTE.md`); the real design quantity is building-weeks of high-baseline monitoring, not a building count alone, so the specific number depends on the monitoring duration chosen for the funded data collection
- Comparative evaluation of the four candidate architectures, reported transparently regardless of outcome
- Locked, pre-registered unified episode-detection protocol
- Confirmatory evaluation on a final untouched cohort (precedent: `STEP2C_HOLDOUT_RESULTS.md`)

---

## Aim 2: Develop and validate an operational per-deployment calibration protocol

### Rationale and preliminary data

A probability calibrator (Platt scaling) fit on the development cohort doesn't transfer cleanly to an independent external cohort. It over-corrects, landing at a calibration slope of 1.165 against an ideal of 1.0, rather than generalizing, while refitting locally on the target cohort restores near-ideal calibration (slope 1.002) (`CALIBRATOR_TRANSFER_TEST.md`). In other words, the model's discrimination (its ranking ability, reflected in AUROC and AUPRC) may transfer reasonably well across deployments, but its calibration (a literal probability or risk-percentage claim) doesn't. That distinction matters for any product that reports numeric risk to end users.

A first-pass simulation of how much per-property data a new deployment needs before local recalibration stabilizes was inconclusive (`RECALIBRATION_DATA_REQUIREMENT.md`), but pointed to something in the range of 60 to 180 days, or roughly 30 to 130 positive alert windows, with a clear methodological fix (a fixed evaluation window) identified for pinning this down properly.

### Specific approach

**Design and validate the deployment break-in protocol.** New deployments start on the shared model's raw discriminative ranking (threshold-based alerting) rather than literal calibrated risk percentages, since ranking-based alerting doesn't need calibration to already be correct. We'll validate that this break-in mode gives an acceptable alert burden (false alarms per property-day) using the same event-level methodology as the preliminary work.

**Rigorously determine the data-sufficiency threshold for local recalibration.** Repeat the recalibration simulation with the methodological fix already identified (a fixed-size held-out evaluation window at each training-budget step, rather than "all remaining data," which confounds the comparison as the training window grows), across a larger and more diverse set of properties: both existing cohorts plus Aim 1's expanded high-baseline dataset. The deliverable is a validated data-sufficiency gate, expressed in accumulated positive-alert-windows rather than calendar time, since the preliminary data suggest episode count, not elapsed time, is what actually limits this.

**Build and validate ongoing calibration-drift monitoring.** Real deployments will see seasonal shifts in ambient air quality and changing occupant behavior. We'll design and validate a monitoring mechanism, such as a rolling-window calibration-slope check, that flags when a property's calibration has drifted enough to need a refresh, and validate the refresh procedure with the same held-out methodology.

### Milestones

- Validated break-in-mode alert-burden characteristics across both existing cohorts
- A rigorous, fixed-window recalibration data-sufficiency threshold, reported as a specific number
- Validated drift-detection and refresh mechanism
- End-to-end operational protocol specification, ready to integrate into the CleanStay AI deployment pipeline
