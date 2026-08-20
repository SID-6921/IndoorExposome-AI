# Hybrid episode definition (single first-pass run)

**One clean run only, as requested: early exploration, not a validated result.** Same pipeline as Step 2B (Full+ features, logistic regression, 5-fold group-CV, 45-min primary horizon, ≤2 FA/home-day operating point, 30-min alert cooldown), with the episode label changed to the hybrid rule: **PM2.5 ≥ min(35 µg/m³, per-home p90-with-12µg-floor), sustained ≥15 min.**

| | Primary (fixed 35) | Sensitivity (per-home relative) | **Hybrid (union)** |
|---|---|---|---|
| Qualifying episodes | 130 | 245 | **250** |
| Per-minute AUPRC / AUROC | 0.0415 / 0.742 | 0.0592 / 0.725 | **0.0571 / 0.722** |
| Event sensitivity @ ≤2 FA/day | 59.2% | 45.3% | **41.2%** |
| Median first-alert lead | 16.0 min | 12.0 min | **12.0 min** |
| Alert precision | 10.0% | 14.2% | **13.6%** |

**Honest headline: the hybrid definition does not clear the benchmark in this first pass, and it doesn't simply inherit the best properties of the two definitions it combines.** Event sensitivity (41.2%) is actually the *lowest* of the three, lower than the per-home-relative definition alone (45.3%), despite recovering nearly the same episode count (250 vs. 245) plus the 10 high-baseline-only episodes the relative definition misses. Lead time (12.0 min) matches the relative definition, still below the 15–20 min bar. Alert precision is the best of the three (13.6%), a modest positive.

**Why detection might drop rather than average out:** the union set pools episodes from very different regimes. The 10 fixed-only episodes (moderate severity, short duration, from already-dirty homes) sit alongside the 123 relative-only episodes (milder absolute concentration, longer duration, from clean homes) and the 120 shared big events. A single model trained to detect all three regimes at once may not be well-suited to any of them individually. This looks like exactly the kind of "characterize the edge cases, don't just union the thresholds" work that would be the real substance of a funded Aim 1, not something a first-pass model was likely to solve outright.

**This is early exploration suggesting the hybrid-definition direction is a real, open technical question, not yet a validated improvement.** Consistent with treating Aim 1 as substantive future work rather than something already solved.
