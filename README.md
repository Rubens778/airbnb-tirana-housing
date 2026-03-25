# Airbnb and Housing Affordability in Tirana, Albania

> **Key finding:** A one standard deviation increase in monthly Airbnb 
> listing growth is associated with a **€15.16/month increase in long-term 
> rents** (p=0.017). Entire-home listings captured a **+16.3% price premium** 
> post-2022 tourism boom (p<0.001). Effect is citywide — also present in 
> suburban rents (β=€10.91, p=0.008).

A quantitative study of short-term rental platform effects on housing 
affordability in Tirana.

---

## Key Results

| Model | β | p-value | Significant |
|-------|---|---------|-------------|
| First-differences OLS | +€15.16/month rent | 0.017 | Yes |
| Panel fixed effects | +€1.14/listing | <0.001 | Yes |
| Log-price × room type | +16.3% entire-home premium | <0.001 | Yes |

Robustness: 5/6 alternative specifications confirm baseline findings.

---

## Repository Structureairbnb-tirana-housing/
├── data/
│   ├── raw/                    # Scraped source files
│   │   ├── airbnb_tirana.csv
│   │   ├── google_trends.csv
│   │   ├── numbeo_rents.csv
│   │   ├── bank_of_albania_index.csv
│   │   ├── instat_tourism.csv
│   │   └── str_timeseries_panel.csv
│   └── processed/              # Cleaned analysis files
│       ├── airbnb_clean.csv
│       ├── main_panel.csv
│       └── neighborhood_features.csv
├── notebooks/
│   └── airbnb_tirana_analysis.ipynb
├── outputs/
│   ├── figures/                # 10 publication figures
│   ├── model_results.csv
│   └── robustness_checks.csv
├── docs/
│   └── DOCUMENTATION.md
├── README.md
├── requirements.txt
└── .gitignore

---

## Replication
```bashgit clone https://github.com/YOURUSERNAME/airbnb-tirana-housing
pip install -r requirements.txt
jupyter notebook notebooks/airbnb_tirana_analysis.ipynb

No API keys required. Runtime approximately 15 minutes.

---

## Data Sources

| Dataset | Rows | Source | Period |
|---------|------|--------|--------|
| Airbnb listings | 661 | Airbnb internal API | March 2026 |
| STR time series | 11,874 | Review archaeology | 2018–2025 |
| Rent index | 11 years | Numbeo | 2015–2025 |
| Property price index | 41 quarters | Bank of Albania | 2015–2025 |
| Tourism arrivals | 87 months | INSTAT Albania | 2018–2025 |
| STR demand proxy | 87 months | Google Trends | 2018–2025 |

---

## Citation
author = Rubens Qosja,
in Tirana, Albania,
note   = Working Paper,
year   = 2026
