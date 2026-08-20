# Recalibration data-requirement simulation

**One run, hedged, and the honest result is that this first-pass design doesn't give a clean answer.** Simulated a "new deployment" using each of the 100 homes' own chronological data: fit a Platt calibrator on the first N days, evaluate its slope on that same home's remaining (held-out, later) days, swept N = 7/14/30/60/90/180/270 days, aggregated across homes.

| Training budget | Median calibration slope | Homes with valid fit | Median training-set positive windows |
|---|---|---|---|
| 7 days | −0.05 | 48 | 6 |
| 14 days | 0.48 | 60 | 10 |
| 30 days | 0.66 | 71 | 17 |
| 60 days | 0.73 | 79 | 29 |
| 90 days | 0.71 | 82 | 47 |
| 180 days | **1.04** | 85 | 130 |
| 270 days | 0.51 | 72 | 224 |

**This is not a clean monotonic convergence, and that itself is the honest finding.** Slope trends toward ideal (1.0) through 60–180 days, coming closest at 180 days, but then drops back to 0.51 at 270 days rather than continuing to stabilize. That drop is very likely a **methodological artifact, not a real degradation**: at 270-day training budgets, the held-out "remaining days" test window shrinks to under 100 days for a full-year home, and the set of homes with enough data to even attempt this shrinks too (72 homes vs. 85 at 180 days). Both changes bias which homes and which time periods are being averaged. A cleaner design would fix the *test window* size (e.g., always evaluate on the next 60 days, not "all remaining days") so the comparison across budgets isn't confounded by a shrinking, changing-composition test set.

**Rough, hedged takeaway for the commercial story:** the data suggest somewhere in the **60–180 day / roughly 30–130 accumulated positive-alert-windows** range before calibration starts looking reasonable, with 180 days being the best point observed here, but this first pass isn't rigorous enough to commit to a specific number for the aims page. A proper version of this analysis (fixed test window, more bootstrap repeats per home, likely restricted to homes with enough total data to support all budgets fairly) would be appropriate scoped work for Aim 2/3 rather than something to finalize now.
