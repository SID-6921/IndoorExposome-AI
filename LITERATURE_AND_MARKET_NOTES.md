# Literature & Market Notes (Item 6)

Brief pass per your instruction, not exhaustive. **All 5 citations below have now been individually verified against authoritative sources (NCBI MEDLINE/PubMed E-utilities, publisher/DOI resolution): every one checks out, and one real error was caught and fixed (#3).**

## Indoor PM2.5 health burden (for Significance)

1. **World Health Organization. (2021). *WHO global air quality guidelines: particulate matter (PM2.5 and PM10), ozone, nitrogen dioxide, sulfur dioxide and carbon monoxide.* Geneva: World Health Organization. ISBN 9789240034228.** Verified directly on who.int. Establishes the benchmark thresholds (5 µg/m³ annual, 15 µg/m³ 24-hour) grants typically cite to frame "safe" exposure, directly relevant to the concentration your system predicts excursions above.
2. **National Academies of Sciences, Engineering, and Medicine. (2024). *Health Risks of Indoor Exposure to Fine Particulate Matter and Practical Mitigation Solutions.* Washington, DC: The National Academies Press. doi: 10.17226/27341.** Verified directly on nationalacademies.org: exact title, year, and committee (chair Richard L. Corsi) confirmed. Consensus report specifically on indoor PM2.5, the most directly on-topic and recent anchor citation available.
3. **Di, Q., Wang, Y., Zanobetti, A., Wang, Y., Koutrakis, P., Choirat, C., Dominici, F., & Schwartz, J.D. (2017). "Air Pollution and Mortality in the Medicare Population." *New England Journal of Medicine*, 376(26), 2513–2522. doi: 10.1056/NEJMoa1702747. PMID: 28657878.** **Correction:** verified against PubMed and found the original draft citation included a fabricated co-author ("Zeger, S.L.") who is not on the real author list; corrected above to the actual 8 authors. Landmark 60.9M-person cohort showing mortality risk from PM2.5 even below EPA/WHO standards, the standard "no safe threshold" citation.
4. **Gutiérrez-Avila, I., Riojas-Rodríguez, H., Colicino, E., Rush, J., Tamayo-Ortiz, M., Borja-Aburto, V.H., & Just, A.C. (2023). "Short-term exposure to PM2.5 and 1.5 million deaths: a time-stratified case-crossover analysis in the Mexico City Metropolitan Area." *Environmental Health*, 22, Article 70. doi: 10.1186/s12940-023-01024-4. PMID: 37848890.** Verified against PubMed: full author list and journal (published version, not the earlier medRxiv preprint) confirmed. Addresses *acute/short-term* PM2.5 exposure and mortality specifically, the best match for an "episode," not chronic-exposure, framing.
5. **Ndlovu, N., & Nkeh-Chungag, B.N. (2024). "Impact of Indoor Air Pollutants on the Cardiovascular Health Outcomes of Older Adults: Systematic Review." *Clinical Interventions in Aging*, 19, 1629–1639. doi: 10.2147/CIA.S480054. PMID: 39372166, PMCID: PMC11453128.** 38-study review (Dept. of Biological and Environmental Sciences, Walter Sisulu University, South Africa) on indoor air pollutant exposure and cardiovascular outcomes. Authorship verified directly against NCBI's MEDLINE record, checks out cleanly, safe to cite.

Not included: a GBD/Health Effects Institute global-burden mortality figure. Didn't verify a specific recent number with confidence, so flagging the gap rather than risking a stale/wrong statistic. Worth adding if you want a global figure and can source one directly.

## Market scan: predictive vs. reactive alerting (for Commercialization Plan)

Checked Awair, Airthings, IQAir/AirVisual, Dyson, PurpleAir (Molekule/Blueair didn't return specific feature data).

**Finding: everything found is reactive, not predictive, at the level CleanStay AI targets.**

- Awair, Airthings, Dyson (MyDyson), PurpleAir: real-time monitoring + threshold-crossing notifications (alert fires *after* a reading already exceeds a set level). No evidence of forecasting an indoor episode before it occurs.
- Airthings forecasts *outdoor pollen*, not indoor PM2.5.
- **IQAir/AirVisual is the one partial exception**: it advertises 3-day/7-day "predictive" air quality forecasts, but per their own documentation this is built from outdoor numerical weather prediction (GFS) + government/community monitoring stations at city/regional level, not a personalized, sensor-driven prediction of one room's PM2.5 trajectory 30–60 minutes out. A meaningfully different product; should be described as partial overlap, not a direct competitor.
- Academic literature (LSTM/attention-based 5–30-min-ahead indoor PM2.5 prediction) confirms the underlying technique is validated in research, but no evidence it's reached a commercial product yet.

**Confidence note:** this was a quick scan, not an exhaustive competitor/patent search. Read as "no predictive room-level indoor PM2.5 alerting product identified in this scan," not "no such product exists anywhere." Smaller startups or recent launches could exist that didn't surface here.
