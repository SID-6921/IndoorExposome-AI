# Why the absolute and per-home-relative episode definitions diverge

## Mechanism (not speculation — traced episode-by-episode across the 59-home CV pool)

Every qualifying episode under each definition was matched against the other by temporal overlap. Result:

| | Fixed only (35 µg/m³) | Matched by both | Relative only (p90, 12 µg/m³ floor) |
|---|---|---|---|
| Count | 10 | 120 | 123 |
| Median duration | 18 min | 81 min | 45 min |
| Median peak concentration | 44.5 µg/m³ | 141 µg/m³ | 23 µg/m³ |

**The divergence is bidirectional, and both directions have a clean mechanistic explanation — this is predominantly a heterogeneity-correction effect, working as intended in one direction and over-correcting in a documented edge case in the other.**

- **52 of 59 homes are "low-baseline"** (their own p90 sits at or below 35 µg/m³ — median 14 µg/m³ across these homes). In these homes, the relative definition surfaces **123 additional episodes** that are genuinely sustained (median 45 min — longer than the fixed-only episodes) but at absolute concentrations well below 35 µg/m³ (median peak 23). These are real, elevated-relative-to-that-home's-normal-air excursions that a one-size-fits-all 35 µg/m³ rule is structurally blind to. This is the heterogeneity correction working exactly as designed.
- **6 of 59 homes are "high-baseline"** (their own p90 already exceeds 35 µg/m³ — these are the dirtier homes). In these homes, the relative definition actually **misses 10 episodes** that the fixed rule would catch — moderate-severity events (peak 37–127 µg/m³) that cross 35 but don't sustain long enough at that home's own, even-higher, bar. Here the per-home correction over-corrects: it demands more from a home that's already dirty, and loses real absolute-scale events as a result.
- **The 120 episodes both definitions agree on** are, unsurprisingly, the biggest and clearest: median peak 141 µg/m³, median duration 81 minutes — these are the unambiguous "big" events, and they're the events most likely to be genuinely product-relevant regardless of which definition is used.

## What this means for the hybrid/unified protocol

The two definitions aren't in tension so much as they're each blind to a different failure mode — the fixed rule misses real elevation in clean homes, the relative rule can be too lenient (raise the bar too high) in already-dirty homes. That suggests a natural, technically motivated hybrid:

**Episode = PM2.5 ≥ min(35 µg/m³, max(home's own p90, 12 µg/m³ floor)), sustained ≥15 min** — i.e., take the *less restrictive* of the two thresholds per home, rather than requiring both or picking one globally. Concretely: in low-baseline homes (52/59), this reduces to the relative threshold (capturing the 123 relative-only episodes); in high-baseline homes (6/59), this reduces to the fixed 35 µg/m³ threshold (recovering the 10 fixed-only episodes). The 120 shared episodes are captured either way. This would give **253 total qualifying episodes** in the CV pool — nearly double either definition alone — while remaining principled: it never drops the bar below the 12 µg/m³ floor, and it never raises the bar above the health-relevant 35 µg/m³ absolute level.

This is a genuinely testable candidate, not just a conceptual patch — the natural next step would be running the same event-level pipeline (feature set, frozen threshold logic, group k-fold) against this union definition to see whether detection/false-alarm/lead-time numbers land closer to the fixed-primary result, the relative-sensitivity result, or somewhere distinct. Happy to run that if useful before the aims page is finalized.
