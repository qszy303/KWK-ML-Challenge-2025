# KWK ML Challenge 2025: Prediction & Unpredictability

**Which local economic indicators drive small business lending across U.S. congressional districts?**

📊 **[View the presentation](https://www.canva.com/d/6T0c8JqcdU8-Fzr)**

---

## Overview

Small businesses make up 99.9% of U.S. businesses and employ nearly half the private-sector workforce, but access to capital varies widely by geography. This project merges IRS income and tax statistics with SBA lender activity for 2022, trains tree-based regression models at the congressional-district level, and asks two things: which economic indicators best predict lending, and where does prediction break down?

Self-employment income per filer was by far the most important feature (importance ≈ 0.26, over 4× the next feature). Most districts were predicted closely, but a cluster of Georgia districts was badly mispredicted, which points to drivers the economic data doesn't capture.

## Research Questions

1. Which IRS-reported economic indicators best predict SBA loan activity by district?
2. Which districts are most and least predictable, and what explains the outliers?

## Data

| Dataset | Source | Geography | Year |
|---|---|---|---|
| Individual income tax statistics | IRS Statistics of Income | ZIP code | 2022 |
| ZIP → congressional district crosswalk | HUD Office of Policy Development & Research | ZIP / district | 2022 |
| Lender activity | U.S. Small Business Administration | Congressional district | 2022 |

- **IRS column definitions:** [data dictionary](https://docs.google.com/document/d/1XS6fCfKlTgeAQqn35OoUpKKpxMi3kqk7/edit?usp=sharing&ouid=113066592990900930020&rtpof=true&sd=true)
- **SBA lender activity:** [download](https://docs.google.com/spreadsheets/d/1WxamBEwZinT2BWsp-BIf231S6PY3kc8S/edit?usp=sharing&ouid=113066592990900930020&rtpof=true&sd=true)

> **⚠️ Required before running:** Download the SBA file above (File → Download → .xlsx) and save it to `data/<!-- TODO: filename -->`. The notebook will fail without it.

2022 was chosen because it has the most overlap between the IRS and SBA releases.

## Methodology

1. **Exploratory baseline.** Before working with small-business data, I modeled a Kaggle personal-loan dataset, where credit history was the dominant predictor. That motivated looking for analogous income and credit-adjacent signals at the district level.
2. **Geographic alignment.** The main cleaning challenge was that the IRS data is reported by ZIP code while the SBA data is reported by congressional district. Since district-level SBA data is the scarcer source, I aggregated IRS ZIP-level data up to districts using the HUD crosswalk.
3. **Feature engineering.** I added normalized features such as self-employment income per filer and business income as a share of AGI, alongside the raw IRS fields.
4. **Modeling.** I used an 80/20 train/test split and compared four regressors:

   | Model | Notes |
   |---|---|
   | Random Forest | Primary model |
   | Gradient Boosting | Best overall test performance |
   | Decision Tree | Smallest train–test gap (least overfitting) |
   | K-Nearest Neighbors | Baseline comparison |

   Overfitting was the main modeling problem given the high feature count relative to roughly 435 districts.
5. **Evaluation.** I scored models with R² on the held-out set, ranked feature importances, and examined per-district percent error to find the best- and worst-predicted districts.

<!-- TODO: state the target variable explicitly (e.g. total SBA loan $ per district) and add R² for each model to the table above -->

## Results

### Feature importance (top 5)

| Rank | Feature | Importance |
|---|---|---|
| 1 | Self-employment income per filer | 0.262 |
| 2 | Business income as % of AGI | 0.059 |
| 3 | `N03300`: returns with self-employment retirement plans | 0.053 |
| 4 | `A85530`: additional Medicare tax | 0.050 |
| 5 | `A09600`: alternative minimum tax amount | 0.045 |

Self-employment activity dominates. The next three features are proxies for higher-income self-employed filers.

### Most and least predictable districts

| Most predictable | % error | | Least predictable | % error |
|---|---|---|---|---|
| NY-03, NY-06, NY-14 | −0.00005% | | GA-04, GA-10 | +74.1% |
| OR-02, OR-05 | +0.0002% | | GA-08, GA-12 | −27.8% |
| NY-22 | −0.0004% | | TX-30 | −17.1% |
| CO-02, CO-04, CO-08 | +0.0007% | | GA-05, GA-13 | +14.9% |
| VA-01, VA-05 | −0.0009% | | IA-04, MO-06 | −12.7% |

### Case study: GA-04

GA-04 had the largest error. In 2022 its representative introduced legislation aimed at expanding funding access for disadvantaged small businesses. That is one plausible reason loan activity diverged from what the economic indicators alone would predict. It is a hypothesis, not a tested explanation.

## Limitations

- **Single year (2022).** There is no temporal validation, and the results may not generalize.
- **Small sample.** About 435 districts is thin for flexible models, so overfitting remains a concern.
- **Imperfect geographic mapping.** ZIP codes can span multiple districts, so the crosswalk introduces noise. Several districts share identical error values, which may be an artifact of this aggregation.
- **Omitted variables.** Policy, lender density, and demographics aren't included, and these may matter more than tax-return features.
- **Correlation, not causation.** Feature importance shows predictive association, not drivers of lending.

## Next Steps

- **Policy use:** Flag districts with strong economic indicators but lower-than-predicted lending as candidates for targeted SBA outreach, and compare policy environments across over- and under-predicted districts.
- **Research use:** Add multiple years, lender-level and demographic data, and a cleaner district mapping to test whether the self-employment signal holds.

## Repository Structure

```
├── data/              # place downloaded SBA file here
├── notebooks/
│   └── <!-- TODO -->.ipynb
├── requirements.txt
└── README.md
```

## How to Run

```bash
git clone https://github.com/<!-- TODO: user -->/KWK-ML-Challenge-2025.git
cd KWK-ML-Challenge-2025
pip install -r requirements.txt
# download SBA data into data/ (see Data section)
jupyter notebook notebooks/<!-- TODO -->.ipynb
```
