# Bootstrap CIs on the fixed/relative/hybrid definition comparison

Home-level cluster bootstrap (1000 resamples), same procedure as the LBNL holdout and 100-home external validation. **Two things done here, not just one:** marginal 95% CIs for each definition, and — because marginal CIs alone can be misleading when comparing definitions computed from the same underlying homes — **paired differences using the same bootstrap draw across all three definitions simultaneously**, which correctly accounts for the shared home-level sampling rather than treating each definition's uncertainty as independent.

**Note on point estimates:** rerunning this comparison (fresh CV folds, 58 homes with sufficient valid readings vs. 59 in the earlier single-definition passes) shifted the point estimates slightly from what was reported previously — fixed 59.2%→57.7%, relative 45.3%→45.7%, hybrid unchanged at 41.2%. This run-to-run drift from ordinary CV stochasticity is itself part of why this bootstrap analysis was worth doing — point estimates alone aren't perfectly stable, which is exactly the kind of thing a CI is supposed to capture.

## Point estimates + marginal 95% CIs

| Definition | Event sensitivity | 95% CI | Median lead | 95% CI |
|---|---|---|---|---|
| Fixed (35 µg/m³) | 57.7% | [40.8%, 75.0%] | 18.0 min | [7.0, 21.0] |
| Relative (p90/12µg floor) | 45.7% | [34.2%, 57.8%] | 12.5 min | [8.5, 17.0] |
| Hybrid (union) | 41.2% | [30.4%, 52.5%] | 12.0 min | [9.0, 18.0] |

**Read in isolation, these marginal CIs overlap substantially** — at face value it might look like the three definitions can't be confidently distinguished from each other.

## Paired differences (same bootstrap draw — this is the correct test)

| Comparison | Mean difference | 95% CI | Distinguishable from zero? |
|---|---|---|---|
| Fixed − Relative | +12.5 pp | [−2.1, +27.5] | **No** — CI includes zero |
| Fixed − Hybrid | +17.0 pp | [+2.8, +32.0] | **Yes** — fixed is genuinely higher |
| Relative − Hybrid | +4.6 pp | [+1.6, +8.3] | **Yes** — relative is genuinely higher |

## What this means

**The hybrid (union) definition is confidently worse than both single definitions — that's a real, statistically supported finding, not noise.** Both "Fixed − Hybrid" and "Relative − Hybrid" exclude zero at the 95% level. But **"Fixed vs. Relative" cannot be confidently distinguished** — despite a 12.5-percentage-point point-estimate gap, the paired CI includes zero, meaning this sample can't rule out the two single definitions performing equivalently.

This sharpens the framing for Aim 1: the data don't yet support claiming the fixed definition is definitively superior to the relative one (that comparison stays genuinely open, which is useful — it's not a foregone conclusion). What they do support is that the naive union/hybrid approach tested here is confidently a step backward relative to either single definition alone — a real, useful negative result that narrows the search space for a better unified protocol rather than closing the question.
