# Iowa Colorectal Cancer & Social Determinants of Health Analysis

*Conducted as part of a consulting externship in collaboration with TruBridge Healthcare through Extern.*

## Executive Summary
Colorectal cancer caught early is highly treatable — but getting screened requires access to care. This project examined whether social and economic conditions across Iowa's 99 counties predict who is most at risk for late-stage colorectal cancer diagnosis. Using county-level data from the Iowa Cancer Registry and the U.S. Census Bureau's American Community Survey (2016–2020), three commonly assumed social risk factors were tested: poverty, health insurance coverage, and housing cost burden. The analysis combined choropleth mapping, correlation testing, ANOVA, multivariate regression, and principal component analysis (PCA) to determine which factors actually move the needle — and which don't. The central finding was clear: health insurance access among working-age adults is the only social determinant that consistently and significantly predicts late-stage colorectal cancer risk across Iowa counties.

## Business Problem

TruBridge serves rural and community-based health systems across Iowa — organizations that operate on thin margins where late-stage cancer cases aren't just a clinical problem, they're a financial one. Stage IV colorectal cancer costs 3–5x more to treat than early-stage disease, and rural hospitals disproportionately absorb that burden when patients arrive too late. The challenge is that without county-level data, health systems are making educated guesses about where to direct screening outreach, insurance navigation, and prevention dollars.

Social determinants of health (SDOH) — the economic and social conditions that shape access to care — are widely cited as drivers of health disparities, but the specific factors that matter most in a given region are rarely tested rigorously. Poverty and housing stress are commonly assumed to predict poor health outcomes, but that assumption hadn't been validated at the county level for Iowa. This project was designed to answer the question directly: **which social factors actually predict late-stage colorectal cancer risk across Iowa, and where should TruBridge clients be targeting their resources?**

## Methodology

**1. Data Collection & Preparation**
Three SDOH datasets from the U.S. Census Bureau's ACS 5-Year Estimates (2016–2020) were merged with late-stage colorectal cancer incidence data from the Iowa Cancer Registry. Poverty rates were drawn from ACS Table S1701 across 12 demographic subgroups (overall, child, senior, racial, educational, and employment-based poverty). Health insurance coverage was drawn from ACS Table B27010, broken down by coverage type — private, employer-only, Medicaid-only, and uninsured — separately for working-age adults (35–64) and seniors (65+). Housing cost burden was drawn from ACS Table B25070, covering the share of renters paying more than 30% and more than 50% of income on rent.

**2. Univariate Analysis — Poverty (Notebook 1)**
Pearson correlations were computed between all 12 poverty subgroups and both cancer outcomes across 99 counties. Side-by-side Iowa choropleth maps visualized the geographic relationship between poverty and cancer risk. Counties were split at the median poverty rate (10.9%) and cancer outcome distributions were compared using independent samples t-tests validated with Shapiro-Wilk normality checks. Violin plots and scatter plots with regression lines were used to assess the shape and direction of any relationships.

**3. Univariate Analysis — Health Insurance (Notebook 2)**
The same correlation and mapping approach was applied to insurance variables, with particular attention to the 35–64 age group. Top-10 county bar charts ranked counties by uninsured rate. An age-group comparison scatter — 35–64 vs 65+ — was produced to test whether the insurance–cancer relationship changed once Medicare coverage became near-universal at age 65. ANOVA was used to test whether uninsured rates differed significantly across cancer risk tiers (lower, moderate, higher).

**4. Univariate Analysis — Rent Burden (Notebook 3)**
All three rent burden categories were tested against both cancer outcomes using Pearson correlations and scatter plots with regression lines — six combinations in total. Choropleth maps and violin plots were used to assess geographic and distributional patterns.

**5. Multivariate Analysis (Notebook 4)**
All three SDOH datasets were merged into a single analysis frame and a comprehensive correlation heatmap was generated across all 14 variables. A reduced 9-predictor linear regression model was built using StandardScaler-normalized features to address multicollinearity among insurance variables (private, government, and employer-only insurance sum to ~100%, making them redundant in the same model). Partial regression plots isolated each predictor's effect after controlling for all others. Principal component analysis (PCA) was run on all 12 predictors to identify latent county profiles. Geographic residual maps were produced showing where the SDOH model explained cancer risk well and where unexplained elevated risk remained.

## Skills

- Data wrangling and multi-source dataset merging (Pandas, NumPy)
- Geospatial analysis and choropleth mapping (GeoPandas, U.S. Census TIGER/Line)
- Statistical testing: Pearson correlation, t-tests, ANOVA, Shapiro-Wilk normality
- Multivariate regression with multicollinearity detection and correction
- Dimensionality reduction and PCA biplot visualization (scikit-learn)
- Partial regression analysis to isolate individual predictor effects
- Data visualization: scatter plots, violin plots, heatmaps, bar charts, residual maps (Matplotlib, Seaborn)
- Translation of statistical findings into plain-language business recommendations

## Results

- **Insurance access is the only significant SDOH predictor.** Private insurance coverage among 35–64 year olds is the strongest protective factor (r = −0.278, p = 0.005). Higher uninsured rates significantly predict higher cancer risk (r = +0.199, p = 0.048). Both findings held up across univariate testing, ANOVA, and multivariate regression.

- **The Medicare effect confirms the mechanism.** Among 35–64 year olds (pre-Medicare), uninsured rate significantly predicts cancer risk. Among 65+ year olds (post-Medicare), the relationship disappears entirely (r = +0.082, p = 0.37). Universal coverage at 65 eliminates the disparity — proving that access to insurance, not some other unmeasured factor, is what's driving the gap.

- **Poverty does not predict cancer risk at the county level in Iowa.** All 12 poverty subgroups were non-significant. High-poverty and low-poverty counties have essentially identical distributions of cancer outcomes. Directing cancer prevention resources based on poverty maps would miss the counties that actually need them.

- **Rent burden shows no relationship with cancer outcomes.** Six combinations tested, all correlations near-zero (r = −0.055 to +0.015), all p > 0.59. Iowa's relatively affordable housing market does not create the cancer risk disparities seen in higher-cost regions.

- **PCA identifies three distinct county profiles.** An insurance/poverty axis explains 45.6% of county-level variation, a housing stress axis explains 17.7%, and a senior vulnerability axis explains 9.5%. These profiles confirm that insurance and poverty are distinct from housing as a driver — and that insurance is the dominant axis.

- **Geographic residuals identify counties with unexplained high risk.** The SDOH model (R² ≈ 0.11) explains a meaningful but partial share of cancer variation. Red-residual counties — where actual risk exceeds what SDOH predicts — point to unmeasured drivers like rurality, screening access, and behavioral factors as priorities for future data collection.

## Notebooks

| Notebook | Focus | Key Output |
|---|---|---|
| `notebook1_poverty.ipynb` | Poverty vs cancer outcomes | Exported poverty SDOH CSV |
| `notebook2_health_insurance.ipynb` | Insurance vs cancer outcomes | Exported insurance SDOH CSV |
| `notebook3_rent_burden.ipynb` | Rent burden vs cancer outcomes | Exported rent burden SDOH CSV |
| `notebook4_multivariate.ipynb` | All factors combined | Regression, PCA, residual maps |

> All notebooks run in **Google Colab**. Run notebooks 1–3 before notebook 4. Each exports a CSV that the multivariate notebook requires.

```python
!pip install geopandas  # only non-standard dependency
```

## Interactive Dashboard

🔗 **[Explore cancer risk and SDOH factors county by county across Iowa](https://claude.ai/public/artifacts/0e5ac430-e52c-4041-8eee-7122405756c2)**
*TruBridge Healthcare Data Analytics Externship · Sriram Srinivasan (June 2025 cohort)*
