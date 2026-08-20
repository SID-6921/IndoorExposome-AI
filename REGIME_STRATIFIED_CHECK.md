# Regime-stratified feasibility check (item 5, optional)

**One clean run, hedged — a sanity check on the premise, not a validated result.** Compared the pooled hybrid-definition model (scored separately on each regime's episodes) against models trained exclusively within each regime: low-baseline (52 homes, 191 episodes, standard 5-fold group-CV) and high-baseline (6 homes, 59 episodes, leave-one-home-out CV given the small n).

| | AUROC | Event sensitivity | Median lead |
|---|---|---|---|
| Pooled model, scored on low-baseline homes | 0.739 | 45.0% | 13.0 min |
| **Low-baseline-only model** | 0.748 | **43.5%** | 12.0 min |
| Pooled model, scored on high-baseline homes | 0.615 | 13.6% | 16.5 min |
| **High-baseline-only model** | 0.613 | **18.6%** | 19.0 min |

**Mixed result, not a clean "yes."**

- **Low-baseline homes (the large majority, 52/58):** regime-specific training does not beat the pooled model — if anything the pooled model is marginally ahead (45.0% vs. 43.5%), a difference almost certainly within noise. No signal that separating helps here.
- **High-baseline homes (a small minority, 6/58):** regime-specific training gives a modest edge (13.6%→18.6% detection, 16.5→19.0 min lead), but with only 6 homes and 59 episodes via leave-one-home-out CV, this comparison itself is data-starved — the improvement could easily be noise, and both numbers remain far below any usable detection threshold regardless.
- **Worth flagging on its own, independent of the regime-training question:** the pooled hybrid model's overall 41.2% sensitivity (reported earlier) is doing much better on the 52 low-baseline homes (45.0%) than on the 6 high-baseline homes (13.6%) — the aggregate number was masking a real, large gap between regimes. That gap exists whether or not regime-specific training turns out to help.

**Read as:** a genuine open question, not a clear signal either for or against a regime-aware direction. The high-baseline regime looks worth more investigation (there may be something there, but 6 homes isn't enough data to know), while the low-baseline regime doesn't show any obvious benefit from separating in this first pass.
