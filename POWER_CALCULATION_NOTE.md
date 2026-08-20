# Power/precision calculation for Aim 1's high-baseline-regime sample size target

The "25 to 30 high-baseline buildings" figure in the original Research Strategy draft was not backed by a calculation. It was a round number that sounded reasonable. Flagging that plainly, per the PI's request, rather than reverse-engineering a justification for it.

## What an actual calculation shows

A standard precision-based sample size formula for estimating a proportion (n = z²·p(1-p)/E², z = 1.96 for 95% confidence) was run at the two observed high-baseline effect sizes (13.6% in LBNL, 68.0% in the 100-home cohort) and a mid-range value (45%), at three target margins of error:

| p | margin ±10pp | margin ±15pp | margin ±20pp |
|---|---|---|---|
| 0.136 | 45 episodes | 20 episodes | 11 episodes |
| 0.450 | 95 episodes | 42 episodes | 24 episodes |
| 0.680 | 84 episodes | 37 episodes | 21 episodes |

That's the naive version, treating each episode as an independent observation. It isn't: episodes within the same building are correlated, so the real precision is worse than this table suggests. Rather than assume a design effect, one can be estimated directly from data already reported. The LBNL holdout's observed sensitivity (6 of 10 episodes detected, across 10 homes) had a home-level cluster-bootstrap 95% CI of [6.3%, 93.3%] (width 0.870), reported in `STEP2C_HOLDOUT_RESULTS.md`. A naive (non-clustered) Wilson interval for the same 6/10 result would be [0.313, 0.832] (width 0.519). The ratio of those widths, 1.68, is an empirical design effect for this specific outcome and clustering structure, not an assumed textbook value.

Applying that factor, the effective (clustering-adjusted) episode counts needed for a ±15pp margin at the mid-range effect size come to roughly 42 × 1.68 ≈ 71 episodes.

## Why a single building count still isn't defensible

Episode yield depends jointly on the number of buildings and how long each is monitored, and the two existing cohorts differ enormously on the second factor: LBNL homes were monitored about a week each and yielded roughly 9.8 high-baseline episodes per building per week; the 100-home cohort ran a full year and yielded roughly 7.7 per building per week. Those per-week rates are reassuringly close to each other, which supports extrapolating from them, but they mean the real design quantity is building-weeks of high-baseline monitoring, not a building count alone. Roughly 70 effective episodes could come from 10 buildings monitored a week each, or from 2 to 3 buildings monitored a month each, or any combination in between.

## Recommendation

Replace the specific "25 to 30 buildings" milestone with the PI's own suggested wording: a target sample size to be finalized via power analysis, informed by the effect sizes observed in the preliminary regime comparisons, once the planned monitoring duration per building for the funded data collection is set. The building-weeks framing above, and the empirical 1.68x design effect, are ready to go into that final calculation once duration is decided.
