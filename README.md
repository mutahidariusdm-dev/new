# Aviation Accidents Analysis

Analysis of U.S. civil aviation accident data (1948-2023) to identify airplane makes/models
with low rates of aircraft destruction and low likelihood of fatal/serious passenger injury,
prepared for an airline/airplane insurer client evaluating fleet risk.

## Business Understanding

**Stakeholder:** An airline/airplane insurer deciding which aircraft makes and models to
favor (and underwrite) based on historical accident outcomes.

**Key questions:**
1. Which airplane makes/models exhibit low rates of total aircraft destruction and low
   likelihood of fatal/serious passenger injury in an accident?
2. Do these recommendations differ for small vs. larger passenger aircraft?
3. What other conditions (weather, engine configuration, phase/purpose of flight, etc.)
   are associated with worse outcomes?

**Scope constraint:** the client only cares about aircraft that could plausibly still be
active today, so the analysis is restricted to professional (non-amateur) builds from
accidents occurring in **1983 or later** (assuming a 40-year maximum aircraft lifetime).

## Data Understanding and Analysis

**Source:** NTSB aviation accident data, 1948-2023 (Kaggle:
[`mos3santos/acidentes-de-aviao-at-2023`](https://www.kaggle.com/datasets/mos3santos/acidentes-de-aviao-at-2023)).
90,000+ raw records covering all aircraft categories; this project filters to **airplanes only**.

**Files in this repo:**
| File | Description |
|---|---|
| `Aviation_Accidents_Cleaning_final.ipynb` | Loads the raw NTSB dataset, inspects missingness, cleans and filters it down to post-1983 airplanes with reasonably-represented makes (n >= 50), and engineers derived fields (`total_passengers`, `fatal_injury_rate`, `aircraft_damage`, `plane_type`, `plane_size`). Outputs `aviation_data_aeroplanes_accidents.csv`. |
| `Aviation_Accidents_Data_Analysis.ipynb` | Loads the cleaned data, splits aircraft into small (<= 20 occupants on the accident flight) vs. larger (> 20) groups, ranks makes/models by injury and destruction rates, and analyzes two additional risk factors (weather condition, number of engines). |
| `data/AviationData.csv` | Raw source data (fallback copy, used if a Kaggle download isn't available in the environment running the notebook). |
| `aviation_data_aeroplanes_accidents.csv` | Cleaned, analysis-ready dataset produced by the cleaning notebook. |

**Data preparation highlights:**
- Dropped columns that were >30% missing and not essential to the analysis (kept
  `Aircraft.Category` and `Broad.phase.of.flight` despite high missingness, since they were
  needed for filtering / requested by the client).
- Filtered to `Aircraft.Category == "Airplane"` and accident year >= 1983.
- Verified that the `Fatal(n)` counts embedded in `Injury.Severity` matched
  `Total.Fatal.Injuries` before simplifying the label to `"Fatal"`.
- Treated missing injury/uninjured counts as zero occupants in that category (not "unknown").
- Dropped rows with unusable `"Unknown"` values in `Aircraft.damage` and `Weather.Condition`.
- Kept only makes with >= 50 accidents overall so make-level comparisons are meaningful,
  and only compared groups (make, model, engine count, etc.) with **n >= 10** in any given
  slice, flagging any comparison that fell short of that threshold as a limitation rather
  than silently reporting an unreliable number.
- Built `plane_type` (Make + Model) since `Model` labels are not unique across makes.

**Key derived metric:** `fatal_serious_injury_rate` - the fraction of occupants on a given
accident flight who were fatally or seriously injured (constructed as
`(Total.Fatal.Injuries + Total.Serious.Injuries) / total_passengers`).

### Visualizations (see the analysis notebook for the full set)
1. Side-by-side bar chart: 15 lowest mean fatal/serious injury fraction makes, small vs. large aircraft.
2. Violin plot: distribution of injury fraction for the 10 lowest-risk small-aircraft makes.
3. Bar chart / crosstab: destruction rate and injury rate by weather condition (VMC vs. IMC),
   with a Mann-Whitney U test and chi-squared test confirming statistical significance.

## Findings

**Small aircraft (<= 20 occupants on the accident flight):**
- **Maule, Aviat Aircraft Inc, Boeing (light-aircraft era), Dehavilland, and Bellanca**
  combine comparatively low injury and destruction rates on an adequate sample size (n >= 10).
- At the model level, the **Cessna 172SP, Cessna 195, Piper PA-28R-201**, and similar
  Maule/Diamond/Cessna models had zero or near-zero recorded fatal/serious injuries, though
  on modest sample sizes (10-33 accidents) that should be treated as directional.

**Larger aircraft (> 20 occupants on the accident flight):**
- Large-aircraft accidents are rare in this dataset (144 of 13,460 post-1983 airplane
  accidents), so only 4 makes (**Embraer, McDonnell Douglas, Boeing, Airbus**) have enough
  volume (n >= 10) to compare reliably. **Embraer** and **Boeing** are the best-supported
  recommendations.
- **No individual large-aircraft model** reaches the n >= 10 threshold - we recommend the
  client treat large-aircraft purchasing decisions as make-level, not model-level, pending
  more data.

**Conditions that matter independent of aircraft choice:**
- **Weather:** accidents in poor-visibility/instrument conditions (IMC) are roughly 3x more
  likely to be fatal/serious and to destroy the aircraft than accidents in good visibility
  (VMC) - both differences are highly statistically significant (p < 0.001).
- **Number of engines:** twin-engine aircraft show higher injury and destruction rates than
  single-engine aircraft in this data, but this most likely reflects aircraft class and
  mission profile (twins tend to be larger, faster, and flown in more demanding contexts)
  rather than a direct safety penalty from a second engine - we recommend against using
  engine count alone as a safety filter.

## Conclusion

For the client's fleet risk decision, we recommend prioritizing **Maule, Aviat Aircraft Inc,
Boeing, Dehavilland, and Bellanca** for small aircraft, and **Embraer** or **Boeing** for
larger aircraft, while treating weather-related operational risk (instrument conditions) as
a factor to underwrite/mitigate separately from aircraft choice. The clearest limitation of
this analysis is sample size for larger aircraft: commercial/multi-engine accidents are much
rarer than general-aviation accidents in NTSB data, so large-aircraft findings here should be
treated as a starting point for further due diligence rather than a final answer.

## How to Reproduce

1. `pip install pandas numpy matplotlib seaborn scipy kagglehub jupyter`
2. Run `Aviation_Accidents_Cleaning_final.ipynb` top to bottom. It will try to download the
   raw dataset via `kagglehub`; if that's unavailable (e.g. no Kaggle credentials), it falls
   back to the local copy at `data/AviationData.csv`. This produces
   `aviation_data_aeroplanes_accidents.csv`.
3. Run `Aviation_Accidents_Data_Analysis.ipynb` top to bottom.
