## Why does RF/XGBoost clear the benchmark at 60 min but not 45 min?

The same 130 qualifying episodes exist in the CV pool regardless of horizon — the episode definition (35 µg/m³, ≥15 min) doesn't depend on the forecasting horizon. What changes between 45 and 60 min is the *matching window* used to credit an alert to an episode: an alert only counts as a "hit" for a given episode if it fires within `horizon` minutes before that episode's onset. Widening that window from 45 to 60 minutes has two direct, largely mechanical effects, not evidence of a qualitatively stronger signal at 60 min:

1. **More opportunity for a pre-existing alert to count.** At RF's 60-min operating point, 91/130 episodes (70.0%) have *some* alert in the 60 minutes before onset, vs. 81/130 (62.3%) in the tighter 45-min window. Widening the matching window mechanically increases the chance that an alert already fired earlier — from the same underlying alerting behavior — falls inside it.
2. **Alerts fired further in advance are now eligible to be counted at all.** At 45 min, an alert fired 50 minutes before onset would be invisible to the matching logic (outside the window); at 60 min, it becomes eligible and pulls the median lead time up. This is largely a definitional consequence of the horizon parameter, not a sign the model reasons further ahead at 60 min than at 45 min.

RF: 45 min → 62.3% detection / 14.0-min median lead; 60 min → 70.0% / 20.0-min (both at ~1.9 FA/home-day). XGBoost shows the same directional pattern (50.8%/11.5 min → 56.2%/13.0 min) but never reaches the ≥15-min lead bar at either horizon — its underlying discrimination is weaker, so widening the window helps less.

**Bottom line:** the horizon-60 result should be read as "the same model, given a wider credit-matching window," not as "a materially better model at 60 minutes." This is worth stating plainly in the narrative rather than implying 60-min unlocks new predictive power.
