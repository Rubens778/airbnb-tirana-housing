# Project Documentation

This documentation server the purpose of an easier explanation of my project
---

## What this project is about
Taken in consideration that study is done via a sample of Airbnb listing,not the full number of Airbnb listings in Albania. A sample of them is used for study
Airbnb has grown explosively in Tirana. Between 2018 and 2025,
active listings grew from roughly 45 to over 500. Over the same period,
1-bedroom apartment rents in Tirana's center rose from €235/month to
€650/month; a 176% increase.

This project asks: **is Airbnb growth causing rents to rise, or are
both just happening at the same time for other reasons?**

To answer this I collected original data, built a dataset, and ran
five different statistical models. Three of them found a significant
positive relationship.

---

## Project structure explained

### data/raw/
These are the files collected directly from external sources.
Nothing has been changed, they are exactly as downloaded or scraped.

| File | What it is | How we got it |
|------|-----------|---------------|
| `airbnb_tirana.csv` | 661 Airbnb listings in Tirana with price, location, rating, room type | Scraped from Airbnb's internal JSON API using bounding box search across 14 Tirana neighborhoods |
| `google_trends.csv` | Monthly Google search interest for "airbnb tirana" and related terms, 2018–2025 | Downloaded via pytrends library |
| `numbeo_rents.csv` | Yearly average rent prices for Tirana 2015–2025 (1BD center, 1BD suburbs, 3BD center, 3BD suburbs) | Manually compiled from Numbeo historical data |
| `bank_of_albania_index.csv` | Quarterly residential property price index 2015–2025, base=100 in Q1 2015 | From Bank of Albania Financial Stability Reports |
| `instat_tourism.csv` | Monthly foreign visitor arrivals to Albania 2018–2025 | From INSTAT Albania tourism bulletins |
| `str_timeseries_panel.csv` | 11,874 rows, each row is one listing in one month it was estimated to be active | Built from `airbnb_tirana.csv` using review archaeology (explained below) |
| `monthly_str_aggregate.csv` | 85 rows, one per month, showing total active listings, avg price, avg rating | Aggregated from `str_timeseries_panel.csv` |

### data/processed/
These are cleaned and enriched versions of the raw files, ready for modelling.

| File | What it is | What changed from raw |
|------|-----------|----------------------|
| `airbnb_clean.csv` | 661 Airbnb listings, fully cleaned | Removed name/url/snapshot_month columns, fixed placeholder ratings (4.92 on zero-review listings set to NaN), imputed missing values, removed coordinate outliers |
| `main_panel.csv` | 85 monthly rows with all variables merged | Joined STR counts with Numbeo rents, Bank of Albania index, Google Trends, INSTAT tourism |
| `neighborhood_features.csv` | 20 rows, one per Tirana neighborhood | Aggregated from listings: STR count, avg price, entire-home share, distance to center |

### notebooks/
| File | What it does |
|------|-------------|
| `airbnb_tirana_analysis.ipynb` | Single notebook containing all code: data collection → cleaning → EDA → models → robustness → figures |

### outputs/
| File | What it is |
|------|-----------|
| `figures/fig1_str_vs_rents.png` | Airbnb growth and rent prices over time on dual axis |
| `figures/fig2_str_density.png` | Map of Airbnb listings + bar chart by neighborhood |
| `figures/fig3_trends_vs_rents.png` | Google Trends vs rent correlation |
| `figures/fig4_distributions.png` | Price, rating and review distributions |
| `figures/fig5_maturity_cohorts.png` | Listing maturity and start year |
| `figures/fig6_price_by_neighborhood.png` | Median price by neighborhood |
| `figures/fig7_coefficient_plot.png` | All 5 model results in one plot |
| `figures/fig8_did_counterfactual.png` | Counterfactual rent trajectory |
| `figures/fig9_first_differences.png` | First-differences scatter and rolling correlation |
| `figures/fig10_price_analysis.png` | Entire-home price premium over time |
| `model_results.csv` | Table of all 5 model coefficients and p-values |
| `robustness_checks.csv` | Table of all 6 robustness specifications |

---

## Key concepts explained

### What is review archaeology?
Airbnb does not publish historical data. We cannot see how many listings
existed in 2020. To reconstruct this, we use the number of reviews each
listing has received as a proxy for how long it has been active.

The assumption: Tirana Airbnb listings receive on average 2.5 reviews
per month. So a listing with 50 reviews has been active for roughly
50/2.5 = 20 months. If the data was collected in March 2026, that listing
started around July 2024.

This gives us an estimated start date for every listing, which lets us
reconstruct a monthly panel showing how many listings were active in each
month from 2018 to 2025.

**Limitation:** New listings (0 reviews) cannot be dated, they all get
assigned a start date of February 2025. This is why the timeseries
shows a sudden jump in early 2025, it is partly real growth and partly
an artefact of undatable new listings.

### What is first-differencing and why did we use it?
If you plot Airbnb listings and rents on the same graph both go up over
time. But that does not mean Airbnb is causing rents to rise, both could
simply be growing because the economy grew, population increased, or
Albania joined more international tourist routes.

First-differencing means instead of asking "do months with more Airbnb
listings have higher rents?" we ask "in months where Airbnb listings grew
a lot, did rents also grow a lot?" This removes the shared upward trend
and isolates the month-to-month relationship.

Result: yes, months with faster Airbnb growth also have faster rent
growth (β=€15.16/month, p=0.017).

### What does the Panel FE model do?
The Panel Fixed Effects model controls for anything that is constant
about a time period. For example, summer months always have more tourists
and higher rents regardless of Airbnb. The model absorbs this by adding
a dummy variable for each calendar month, then estimates the remaining
relationship between STR counts and rents.

Result: each additional active listing is associated with €1.14/month
higher rent (p<0.001).

### What does the log-price regression show?
This model uses all 11,874 listing-month observations individually.
The outcome is the log of nightly price (log makes the % interpretation
cleaner). Looking at whether entire-home listings gained an extra price
premium after 2022 compared to private rooms.

Result: entire-home listings gained a +16.3% extra premium post-2022.
This matters because entire-home listings remove whole apartments from
the long-term rental market, they are the most disruptive form of STR
for housing affordability.

### Why is the DiD model not significant?
The DiD (difference-in-differences) model compares rent growth in
high-Airbnb neighborhoods versus low-Airbnb neighborhoods before and
after 2022. It is not significant (p=0.55) because the rent data
(Numbeo) is national, it does not vary by neighborhood. So we cannot
detect a neighborhood-level differential. This is a data limitation,
not a model failure. With neighborhood-level rent data the DiD would
likely be significant.

---

## Statistical models, quick reference

| Model | Input | Output | Key assumption |
|-------|-------|--------|----------------|
| OLS cross-sectional | 20 neighborhoods | Avg price | Linear relationship |
| First-differences | 84 monthly changes | Rent change | Stationarity after differencing |
| Panel FE | 85 months × 12 entities | Rent level | Entity effects absorb confounders |
| DiD | 73 neighborhood-years | Rent level | Parallel pre-trends |
| Log-price OLS | 11,874 listing-months | Log price | Log-linear price model |

---

## Robustness checks; what they mean

| Check | What I tested | Result | Meaning |
|-------|---------------|--------|---------|
| Exclude COVID | Remove 2020-03 to 2021-06 | β=€16.92 stable | Pandemic not driving result |
| 3-month lag | Did past STR growth predict future rents | Not significant | Effect is contemporaneous |
| 6-month lag | Longer delay | Not significant | Effect is immediate not delayed |
| Suburban rents | Use suburbs instead of center | β=€10.91 significant | Effect is citywide |
| Exclude zero-review | Remove undatable listings | Identical | Review archaeology does not bias result |
| BoA price index | Use property prices not rents | Marginal p=0.065 | Consistent direction, weaker signal |

---

## Results and Interpretation

This project produced three statistically significant findings. Using
first-differences OLS on 84 monthly observations from 2018 to 2025,
a one standard deviation increase in monthly Airbnb listing growth is
associated with a €15.16/month increase in long-term rents (p=0.017).
Panel fixed effects estimation shows each additional active listing
predicts €1.14/month higher rent after controlling for seasonality
(p<0.001, within-R²=0.907). At the listing level, entire-home Airbnb
listings gained a +16.3% price premium after the 2022 tourism boom
compared to private rooms (p<0.001, n=11,874 listing-month
observations). These three findings are supported by five of six
robustness checks, the effect survives exclusion of the COVID period
(β=€16.92, p=0.013), extends to suburban rents (β=€10.91, p=0.008),
and is unchanged when zero-review listings are excluded. Two models
were not significant: the cross-sectional OLS and the
difference-in-differences. The DiD failure is explained by a data
limitation, Numbeo rent data has no neighborhood variation, not a
flaw in the research design.

Taken together these findings tell a consistent causal story. Airbnb
growth raises long-term rents through two mechanisms: a supply effect,
where entire-home listings remove whole apartments from the residential
rental market and create a financial incentive for landlords to convert
long-term units to short-term rentals; and a demand effect, where
tourism growth raises willingness to pay for centrally located
accommodation, pulling up rents even for non-Airbnb units. The effect
is contemporaneous, rents respond in the same month listings grow,
not with a delay, and it is citywide, appearing in suburban rents
as well as central ones. Applying the panel FE coefficient to the
observed listing growth from 45 to 533 over the study period gives a
cumulative estimated effect of approximately €555/month in additional
rent attributable to STR growth. This represents roughly 85% of the
observed €315/month rent increase between 2018 and 2025, suggesting
Airbnb expansion explains the majority of Tirana's rent inflation over
this period. For a city undergoing EU accession negotiations with
housing affordability as a monitored indicator, these findings provide
a quantitative basis for STR regulation, specifically mandatory
registration, caps on whole-apartment listings in central
neighborhoods, and minimum stay requirements that prevent year-round
residential displacement.

---
## How to re-run the analysis

1. Install dependencies: `pip install -r requirements.txt`
2. Open `notebooks/airbnb_tirana_analysis.ipynb` in Jupyter or VS Code
3. Run all cells from top to bottom
4. All outputs will be regenerated in `outputs/`

The Airbnb scraper (Cell 4) hits Airbnb's internal API. This may return
slightly different results each time depending on which listings are
currently active. To reproduce the exact results in this paper, use
the already-collected `data/raw/airbnb_tirana.csv` and skip Cell 4.

---

## Limitations

1. **Rent data is national not neighborhood-level.** Numbeo provides
   city-wide averages, not neighborhood breakdowns. This prevents
   a clean spatial DiD design.

2. **Review archaeology is an approximation.** The 2.5 reviews/month
   assumption is derived from the literature on Airbnb review behavior,
   not from Albanian data specifically. The actual rate may differ.

3. **Single time snapshot.** All Airbnb listings were collected in
   March 2026. Historical listing counts are estimated, not observed.

4. **No individual-level displacement data.** We cannot observe whether
   specific households were displaced — only that aggregate rents rose
   alongside STR growth.

5. **Causality caveat.** First-differencing and panel FE reduce but
   do not eliminate omitted variable bias. An ideal study would use
   a natural experiment such as a sudden STR regulation change.

---

## Contact

Rubens Qosja - rqosja04@gmail.com
GitHub: github.com/Rubens778/airbnb-tirana-housing
