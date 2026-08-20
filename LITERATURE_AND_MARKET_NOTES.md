# Literature & Market Notes (Item 6)

Brief pass per your instruction — not exhaustive.

## Indoor PM2.5 health burden (for Significance)

1. **World Health Organization. (2021). *WHO Global Air Quality Guidelines: Particulate Matter, Ozone, Nitrogen Dioxide, Sulfur Dioxide and Carbon Monoxide.* WHO.** Establishes the benchmark thresholds (5 µg/m³ annual, 15 µg/m³ 24-hour) grants typically cite to frame "safe" exposure — directly frames the concentration your system predicts excursions above.
2. **National Academies of Sciences, Engineering, and Medicine. (2024). *Health Risks of Indoor Exposure to Fine Particulate Matter and Practical Mitigation Solutions.* Washington, DC: National Academies Press. doi: 10.17226/27341.** EPA-sponsored consensus report specifically on indoor PM2.5 — the most directly on-topic and recent anchor citation available.
3. **Di, Q., Wang, Y., Zeger, S.L., et al. (2017). "Air Pollution and Mortality in the Medicare Population." *New England Journal of Medicine*, 376(26), 2513–2522.** Landmark 60.9M-person cohort showing mortality risk from PM2.5 even below EPA/WHO standards — the standard "no safe threshold" citation.
4. **Gutiérrez-Avila, I., Riojas-Rodríguez, H., Colicino, E., et al. (2023). "Short-term exposure to PM2.5 and 1.5 million deaths: a time-stratified case-crossover analysis in the Mexico City Metropolitan Area." *Environmental Health*, 22, Article 70.** Addresses *acute/short-term* PM2.5 exposure and mortality specifically — the best match for an "episode," not chronic-exposure, framing.
5. **Systematic review (2024), *Clinical Interventions in Aging* (Dove Medical Press), PubMed ID 39372166 — "Impact of Indoor Air Pollutants on the Cardiovascular Health Outcomes of Older Adults."** 38-study review on indoor air pollutant exposure and cardiovascular outcomes. Note: PubMed listing/abstract confirmed, but full author byline not independently cross-checked — verify before citing.

Not included: a GBD/Health Effects Institute global-burden mortality figure — didn't verify a specific recent number with confidence, flagging the gap rather than risking a stale/wrong statistic. Worth adding if you want a global figure and can source one directly.

## Market scan: predictive vs. reactive alerting (for Commercialization Plan)

Checked Awair, Airthings, IQAir/AirVisual, Dyson, PurpleAir (Molekule/Blueair didn't return specific feature data).

**Finding: everything found is reactive, not predictive, at the level CleanStay AI targets.**

- Awair, Airthings, Dyson (MyDyson), PurpleAir: real-time monitoring + threshold-crossing notifications (alert fires *after* a reading already exceeds a set level). No evidence of forecasting an indoor episode before it occurs.
- Airthings forecasts *outdoor pollen*, not indoor PM2.5.
- **IQAir/AirVisual is the one partial exception** — advertises 3-day/7-day "predictive" air quality forecasts, but per their own documentation this is built from outdoor numerical weather prediction (GFS) + government/community monitoring stations at city/regional level — not a personalized, sensor-driven prediction of one room's PM2.5 trajectory 30–60 minutes out. A meaningfully different product; should be described as partial overlap, not a direct competitor.
- Academic literature (LSTM/attention-based 5–30-min-ahead indoor PM2.5 prediction) confirms the underlying technique is validated in research, but no evidence it's reached a commercial product yet.

**Confidence note:** this was a quick scan, not an exhaustive competitor/patent search. Read as "no predictive room-level indoor PM2.5 alerting product identified in this scan," not "no such product exists anywhere" — smaller startups or recent launches could exist that didn't surface here.
