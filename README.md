# 🏥 Iowa Cancer Risk Analytics — SDOH Intelligence for Rural Health Systems

![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB?logo=python&logoColor=white)
![Pandas](https://img.shields.io/badge/Data-Pandas-150458?logo=pandas&logoColor=white)
![GeoPandas](https://img.shields.io/badge/Geospatial-GeoPandas-139C5A)
![Scikit-Learn](https://img.shields.io/badge/ML-Scikit--Learn-F7931E?logo=scikit-learn&logoColor=white)
![Colab](https://img.shields.io/badge/Runtime-Google_Colab-F9AB00?logo=googlecolab&logoColor=white)
![License](https://img.shields.io/badge/License-Portfolio-lightgrey)

> **Extern × TruBridge Healthcare**  
> A county-level social determinants of health (SDOH) analysis pipeline that tests poverty, insurance access, and rent burden as predictors of late-stage colorectal cancer risk across all 99 Iowa counties — combining choropleth mapping, multivariate regression, and PCA to identify where TruBridge clients should actually be targeting resources.

## 🎬 Demo

[![Interactive Dashboard](https://img.shields.io/badge/Explore-Interactive_Dashboard-00A878?logo=googlecolab&logoColor=white)](https://claude.ai/public/artifacts/0e5ac430-e52c-4041-8eee7122405756c2)

> Click the badge above to explore cancer risk and SDOH factors county-by-county across Iowa in an interactive dashboard.

---

## 📋 Table of Contents

- [Problem Statement](#-problem-statement)
- [What It Does](#-what-it-does)
- [System Architecture](#-system-architecture)
- [Pipeline Walkthrough](#-pipeline-walkthrough)
- [Results](#-results)
- [Tech Stack](#-tech-stack)
- [Design Decisions & Trade-offs](#-design-decisions--trade-offs)
- [Getting Started](#-getting-started)
- [Example Queries & Findings](#-example-queries--findings)
- [Limitations & Future Work](#-limitations--future-work)
- [Project Impact](#-project-impact)

---

## 🎯 Problem Statement

Colorectal cancer caught early is highly treatable — but getting screened requires access to care. TruBridge serves rural and community health systems across Iowa that operate on thin margins. When patients arrive with late-stage disease, the financial impact is compounding.

| Pain Point | How This Analysis Solves It |
|---|---|
| Health systems guessing where to direct resources | County-level SDOH targeting identifies the highest-risk counties by the factor that actually matters |
| Poverty assumed to predict cancer outcomes | Tested rigorously across 12 subgroups — result is counterintuitive and actionable |
| No visibility into insurance coverage gaps | Maps uninsured rates among working-age adults (35–64) county by county |
| Correlation vs. causation | Medicare natural experiment provides near-causal evidence for the insurance mechanism |

---

## ✅ What It Does

1. **Merges** three ACS data tables (S1701, B27010, B25070) with Iowa Cancer Registry incidence data across all 99 counties
2. **Tests** 12 poverty subgroups, 5 insurance coverage types (split by age group), and 3 rent burden categories against two cancer outcomes
3. **Maps** each factor geographically using Iowa county choropleth maps to reveal spatial patterns
4. **Runs** Pearson correlations, t-tests, ANOVA, and scatter regressions for each factor univariately
5. **Combines** all factors into a 9-predictor multivariate regression with multicollinearity correction and StandardScaler normalization
6. **Applies** PCA to identify latent county profiles and explain what drives county-level variation
7. **Produces** geographic residual maps showing where the SDOH model explains cancer risk and where unexplained elevated risk remains
8. **Delivers** three targeted, county-specific recommendations for TruBridge client health systems

---

## 🏗️ System Architecture

```
ACS TABLE S1701          ACS TABLE B27010          ACS TABLE B25070
Poverty Status           Health Insurance           Rent Burden
12 subgroups             5 types × 2 age groups     3 burden categories
       │                        │                          │
       ▼                        ▼                          ▼
┌─────────────────────────────────────────────────────────────────┐
│              Data Cleaning & Feature Engineering                │
│   Custom ACS parsers (suppressed estimates, coded missing)      │
│   Percentage computation per county, multicollinearity check    │
└──────────────────────────────┬──────────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────────┐
│              Iowa Cancer Registry Merge                         │
│   Late-stage colorectal cancer incidence per 100K + risk prob   │
│   99 counties, 2016–2020 window, complete-case analysis         │
└──────────────────────────────┬──────────────────────────────────┘
                               │
          ┌────────────────────┼────────────────────┐
          ▼                    ▼                    ▼
  NOTEBOOK 1             NOTEBOOK 2            NOTEBOOK 3
  Poverty Analysis       Insurance Analysis    Rent Burden Analysis
  Choropleth, violin,    Choropleth, ANOVA,    Scatterplots,
  t-tests, scatter       Medicare proof,       6 combinations,
  (12 subgroups)         top-10 bar charts     all non-significant
          │                    │                    │
          └────────────────────┼────────────────────┘
                               ▼
                        NOTEBOOK 4
               Multivariate Regression + PCA
               9-predictor OLS, feature importance,
               PCA biplots, geographic residual maps
```

### Component Specifications

| Component | Technology | Configuration |
|---|---|---|
| Data Source | ACS 5-Year Estimates (2016–2020) | Tables S1701, B27010, B25070 |
| Cancer Outcomes | Iowa Cancer Registry | AAR per 100K + risk probability |
| Geospatial | GeoPandas + Census TIGER/Line | Iowa county boundaries, 2022 |
| Statistical Testing | SciPy | Pearson r, t-test, ANOVA, Shapiro-Wilk |
| Regression | Scikit-Learn LinearRegression | 9 predictors, StandardScaler normalized |
| Dimensionality Reduction | Scikit-Learn PCA | 12 predictors, full variance decomposition |
| Visualization | Matplotlib + Seaborn | Choropleth, violin, scatter, heatmap, residual maps |
| Runtime | Google Colab | CPU only — no GPU required |

---

## 🔬 Pipeline Walkthrough

**Notebook 1 — Poverty Analysis**  
Extracts 12 poverty subgroups from ACS Table S1701 using row-index parsing and a custom `clean_pct()` function that handles ACS suppressed estimates (parenthetical values), coded missing (`-`, `N`, `X`), and blank fields. Counties are split at the median poverty rate (10.9%) and cancer outcome distributions are compared using t-tests validated with Shapiro-Wilk normality checks. Side-by-side Iowa choropleths and violin plots test whether poverty and cancer risk maps point to the same counties. They don't.

**Notebook 2 — Health Insurance Analysis**  
Extracts coverage variables from ACS Table B27010 by row index for two age groups: 35–64 and 65+. Computes `pct_uninsured`, `pct_private_insurance`, `pct_government_insurance`, `pct_employer_only`, and `pct_medicaid_only` for each group. Top-10 county bar charts surface the highest-uninsured counties. A critical scatter plot overlays both age groups — showing the uninsured-cancer risk trend line for 35–64 year olds and its complete disappearance for 65+ year olds once Medicare coverage becomes near-universal.

**Notebook 3 — Rent Burden Analysis**  
Tests three rent burden categories (affordable <30%, burdened 30–50%, severely burdened 50%+) against both cancer outcomes — six scatter plots with regression lines in total. All six lines are flat. Choropleths confirm that Iowa's rent burden map and cancer risk map point to different counties.

**Notebook 4 — Multivariate Analysis**  
Merges all three SDOH datasets into a single 99-county dataframe. Drops collinear insurance variables (`pct_private_insurance`, `pct_government_insurance`, `pct_employer_only`) — these sum to ~100% with `pct_uninsured`, making them redundant in the same regression. Runs OLS on 9 StandardScaler-normalized predictors. Generates a 14-variable correlation heatmap, standardized coefficient feature importance bars, PCA scree and cumulative variance plots, and three-panel Iowa choropleth maps showing actual cancer risk, model-predicted risk, and residuals county-by-county.

---

## 📊 Results

### Factor Verdicts

| Factor | Direction | r (Cancer Risk Prob.) | p-value | Predictor? |
|---|---|---|---|---|
| Private insurance (35–64) | ↓ Protective | −0.278 | 0.005 | ✅ Significant |
| Uninsured rate (35–64) | ↑ Risk | +0.199 | 0.048 | ✅ Significant |
| Medicaid-only (35–64) | ↑ Elevated risk | — | < 0.05 | ✅ Significant |
| Overall poverty rate | None | ~0 | > 0.05 | ❌ Not a predictor |
| Child & senior poverty (all 12 subgroups) | None | ~0 | > 0.05 | ❌ Not a predictor |
| Rent burden (all 3 categories) | None | −0.055 to +0.015 | > 0.59 | ❌ Not a predictor |

### Multivariate Model

| Metric | Result |
|---|---|
| Strongest positive predictor | `pct_uninsured_35_64` |
| Model R² | ~0.11 |
| Variance explained by PCA PC1 | 45.6% (insurance/poverty axis) |
| Variance explained by PCA PC2 | 17.7% (housing stress axis) |
| Variance explained by PCA PC3 | 9.5% (senior vulnerability axis) |

### Top Target Counties (by uninsured rate, ages 35–64)

| County | Uninsured Rate |
|---|---|
| Davis | 20.1% |
| Ringgold | 12.3% |
| Franklin | 10.9% |
| Van Buren | 10.7% |
| Clarke | 10.5% |

---

## 🛠️ Tech Stack

| Category | Tool |
|---|---|
| Data Wrangling | Pandas, NumPy |
| Statistical Testing | SciPy (Pearson r, t-test, ANOVA, Shapiro-Wilk) |
| Machine Learning | Scikit-Learn (LinearRegression, StandardScaler, PCA) |
| Geospatial | GeoPandas, U.S. Census TIGER/Line shapefiles |
| Visualization | Matplotlib, Seaborn |
| Runtime | Google Colab (CPU) |
| Language | Python 3.10+ |

---

## ⚖️ Design Decisions & Trade-offs

| Decision | Rationale | Trade-off |
|---|---|---|
| Complete-case analysis for multivariate model | Avoids imputation bias in regression | Drops counties with any missing predictor |
| Median imputation for Black poverty (5 counties) | Prevents listwise deletion of sparse rural counties | Slightly smooths rural variation |
| Drop collinear insurance vars in regression | Avoids inflated coefficients from multicollinearity | PCA uses full set — collinearity handled there |
| County-level aggregation | Matches the grain of both ACS and Iowa Cancer Registry data | Ecological fallacy risk — averages mask within-county variation |
| 2016–2020 ACS window | Maximizes sample reliability for small rural counties | Pre-dates post-COVID Medicaid expansion shifts |

---

## 🚀 Getting Started

### Prerequisites
- Google Colab (free tier, CPU runtime works)
- No API keys required

### Run on Google Colab
1. Open any notebook in Google Colab
2. Upload the required CSV files via the folder icon (see note at the top of each notebook)
3. Run all cells sequentially
4. Run notebooks 1–3 before notebook 4 — each exports a CSV that the multivariate notebook requires

### Run Locally

```bash
# Clone the repo
git clone https://github.com/sriramprog/TruBridge-Healthcare-Data-Analytics.git
cd TruBridge-Healthcare-Data-Analytics

# Install dependencies
pip install pandas numpy matplotlib seaborn geopandas scikit-learn scipy jupyter

# Launch
jupyter notebook notebooks/notebook1_poverty.ipynb
```

### Notebook Order

```bash
# Run sequentially — each exports a CSV used by the next
jupyter notebook notebooks/notebook1_poverty.ipynb
jupyter notebook notebooks/notebook2_health_insurance.ipynb
jupyter notebook notebooks/notebook3_rent_burden.ipynb

# Requires the 3 CSVs exported above
jupyter notebook notebooks/notebook4_multivariate.ipynb
```

---

## 🔍 Example Queries & Findings

**Finding #1 — Poverty**
```
Question:  Do high-poverty counties have higher cancer rates?
Test:      t-test splitting counties at median poverty (10.9%)
Result:    Distributions overlap almost entirely — p > 0.05 across all 12 subgroups
Verdict:   ❌ Poverty does NOT predict colorectal cancer risk in Iowa.
           Targeting programs by poverty maps would miss the counties that need them.
```

**Finding #2 — Insurance (The Medicare Proof)**
```
Question:  Does uninsured rate predict cancer risk — and does it hold across age groups?
Test:      Scatter + Pearson r for ages 35–64 vs 65+
Ages 35–64 (no Medicare):  r = +0.199, p = 0.048 → significant upward trend
Ages 65+   (Medicare):     r = +0.082, p = 0.370 → flat, no relationship
Verdict:   ✅ Universal coverage at 65 eliminates the disparity entirely.
           Ages 35–64 is where the gap lives and where intervention has the highest impact.
```

**Finding #3 — Multivariate**
```
Question:  When all 3 factors are combined, which predictor dominates?
Model:     9-predictor OLS, StandardScaler normalized, R² ≈ 0.11
Result:    pct_uninsured_35_64 is the single largest positive coefficient
Verdict:   ✅ Insurance access outweighs poverty and rent burden even in a combined model.
           Rurality, diet, and screening uptake explain the remaining ~89%.
```

> **Note on R²:** An R² of 0.11 is expected at county level with 99 data points and ecological-level predictors. The insurance finding is robust across univariate, ANOVA, and multivariate methods — not an artifact of model fit.

---

## ⚠️ Limitations & Future Work

**Current Limitations**

1. **Ecological Fallacy** — County-level averages mask individual-level variation; a county with 10% average poverty can still contain high-risk pockets
2. **Correlation, Not Causation** — Uninsured rate tracks with cancer risk but direct causation cannot be proven without controlling for all confounders; the Medicare natural experiment provides the strongest available evidence
3. **Missing Variables** — No rurality index, behavioral data (diet, smoking, obesity), or direct screening rate data — the most mechanistically direct link between insurance and cancer outcomes was inferred, not measured

**Roadmap**

| Timeline | Enhancement |
|---|---|
| Short-term | Add rurality index (USDA Rural-Urban Continuum Codes) as a covariate |
| Medium-term | Incorporate BRFSS behavioral data (smoking, obesity, screening rates) at county level |
| Long-term | Extend to all Iowa cancer types and update to 2018–2022 ACS 5-year estimates post-Medicaid expansion |

---

## 🎓 Project Impact

This externship project translated raw public health data into a targeted, data-driven resource allocation framework for rural health systems — demonstrating that the conventional assumption (poverty drives cancer risk) is wrong in Iowa, and that the actionable lever is insurance access among working-age adults.

Key technical skills developed include:

- Multi-source public health dataset merging and ACS variable extraction
- Geospatial analysis and county-level choropleth mapping with Census TIGER shapefiles
- Statistical testing pipeline: Pearson correlation, t-tests, ANOVA, Shapiro-Wilk normality
- Multivariate regression with multicollinearity detection and feature normalization
- PCA for latent county profile identification and dimensionality reduction
- Translation of statistical findings into plain-language business recommendations

Skills directly translatable to healthcare analytics, public health research, and policy roles where the gap between assumed and actual drivers of health disparities has real resource allocation consequences.

---

## 📁 Repository Structure

```
TruBridge-Healthcare-Data-Analytics/
├── notebooks/
│   ├── notebook1_poverty.ipynb           # Factor 1: Poverty — 12 ACS subgroups
│   ├── notebook2_health_insurance.ipynb  # Factor 2: Insurance — ages 35–64 vs 65+
│   ├── notebook3_rent_burden.ipynb       # Factor 3: Rent burden — 6 combinations
│   └── notebook4_multivariate.ipynb      # Combined regression + PCA + residual maps
├── data/
│   └── DATA_DICTIONARY.md                # Variable reference for all ACS tables
├── reports/
│   └── TruBridge_Final_v4.pdf            # Final stakeholder presentation deck
├── README.md                             # This file
└── requirements.txt                      # Python dependencies
```

---
*Extern: Sriram Srinivasan · [GitHub](https://github.com/sriramprog) · [Interactive Dashboard](https://claude.ai/public/artifacts/0e5ac430-e52c-4041-8eee7122405756c2)*
