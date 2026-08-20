# Innovation — Draft

Status: structural draft, built to be tightened into final grant prose.

## The reframing is the innovation, not the model

Logistic regression isn't new, and the stronger baselines tested (Random Forest, XGBoost) are standard off-the-shelf methods. The actual innovation shown in this preliminary work is how the prediction problem is framed and evaluated. There's direct evidence for this from the project's own results, not just an assertion.

The same model, scored two different ways, produced two different conclusions.

Scored with conventional per-minute classification metrics (AUPRC, AUROC, raw false-positive rate), the preliminary models did not clearly or consistently beat a naive persistence baseline (`STEP2_PRELIMINARY_BASELINE_RESULTS.md`). That's the kind of result that normally reads as inconclusive, and it's how most published indoor air quality forecasting work reports results.

Scored instead at the event level, consolidating repeated positive predictions into discrete alert events with a cooldown period, reporting first-actionable-alert lead time rather than the average across all flagged windows, and fixing the false-alarm rate (alerts per property-day) as the operating constraint rather than optimizing a statistical metric like F1, the identical model cleared a defensible operational benchmark: at least 50% episode detection at no more than 2 false alerts per property-day, with a meaningful lead time (`STEP2B_EVENT_LEVEL_CALIBRATION_ANALYSIS.md`).

That gap is the core claim here. Conventional per-minute metrics don't answer the question that decides whether a product is usable: how many real alerts, at what false-alarm cost, with how much real warning. Reframing the evaluation around that question surfaces value the standard reporting convention misses. The reframing itself, along with the operational-benchmark methodology behind it (fixed alert-burden operating points, home-level cluster-bootstrap confidence intervals, a frozen model evaluated once on a sealed holdout), is the transferable contribution. It applies to any sustained-exposure-episode forecasting problem, not just PM2.5 or this particular model family.

## Where this sits relative to existing products

A scan of the major consumer and commercial indoor air quality products (Awair, Airthings, Dyson, PurpleAir, IQAir) found they're all reactive: they show current readings and/or fire an alert once a threshold is already crossed, not before an episode happens (`LITERATURE_AND_MARKET_NOTES.md`).

IQAir/AirVisual needs careful handling rather than either an overclaim against it or lumping it in with the reactive products. IQAir advertises "predictive" air-quality forecasts, but by their own documentation, that's a 3-to-7-day outdoor, city-scale forecast built from numerical weather prediction and government or community monitoring stations. It's a different prediction target: ambient outdoor conditions days out, at city resolution, versus a specific room's indoor PM2.5 trajectory 30 to 60 minutes out, personalized to that space's own sensor history and context. The honest claim is that both use the word "predictive," but no product we found does personalized, room-level, sub-hour indoor exposure forecasting. State it precisely enough that a reviewer who knows IQAir's marketing can't call it an overclaim.

## Two smaller contributions worth naming

Validating across two independently collected cohorts with different sensors, countries, climates, and native time resolutions (1-minute versus hourly) required a documented method for resolution-matched feature construction: aggregating the higher-resolution cohort down to match the lower-resolution one, not the reverse. It also required honestly dropping a feature group the trained model had no equivalent to learn from, rather than forcing it in (`FROZEN_CONFIG_PORTABLE.md`). That's a reusable template for any indoor-sensing product that has to generalize across different hardware.

The frozen-model-before-holdout protocol (`FROZEN_CONFIG.md`, `STEP2C_HOLDOUT_RESULTS.md`) and the discipline of running one clean exploratory pass and reporting it as-is, even on speculative directions like the hybrid definitions and regime-stratified models, is a level of rigor that's uncommon in preliminary data packages, where there's real pressure to keep iterating against a test set until the numbers look better.
