# Regime gap replication test (100-home cohort)

**The LBNL regime gap does not replicate the same way in the larger, independent cohort. If anything, it reverses.** This needs to be reported plainly, since it was being used as the sharper motivator for Aim 1.

Applied the same baseline-regime split (own p90 vs. 35 µg/m³) to all 100 external homes, using the frozen threshold (0.4121) and portable model exactly as deployed: no retraining, no re-tuning.

| | LBNL (6 high / 52 low homes) | 100-home cohort (13 high / 87 low homes) |
|---|---|---|
| Low-baseline event sensitivity | 45.0% | 45.6% |
| High-baseline event sensitivity | **13.6%** (much lower) | **68.0%** (much higher) |
| High-baseline false alerts/home-day | (not separately reported) | **3.33**, well above the ≤2 target |

Low-baseline detection is remarkably consistent across both cohorts (45.0% vs. 45.6%), a reassuring replication of that half of the picture. But the high-baseline regime does not replicate directionally: in LBNL it was the weak point (13.6%, far below low-baseline), while in the 100-home cohort it's actually the stronger one (68.0%), at the cost of a false-alarm rate 60% above target (3.33 vs. the ≤2 goal).

**Read honestly:** the single frozen threshold, calibrated on LBNL's overall pooled distribution, doesn't operate the same way across regimes when applied to a different, more-polluted-on-average cohort. In the 100-home high-baseline homes it fires more often in both directions (more true detections, but also proportionally more false alarms) rather than under-firing the way it did in LBNL's 6 high-baseline homes. This is a different failure mode than the one observed in LBNL, not a confirmation of the same one.

**Implication for Aim 1's framing:** the regime gap as observed in LBNL was built on only 6 homes and does not hold up as a stable, replicating phenomenon at 10x the sample size. It should not be presented as a validated, cross-cohort-consistent motivator. What *does* replicate is that a single fixed threshold does not serve both regimes equally well, a real, consistent finding across both cohorts, just not in the specific direction or magnitude previously assumed. This still supports investigating regime-aware approaches, but the framing needs to shift from "a specific gap that shows up twice" to "the single-threshold approach is consistently regime-sensitive, in ways that differ by cohort." That's a more honest and still-fundable basis, but a different claim than the one currently in the aims page.
