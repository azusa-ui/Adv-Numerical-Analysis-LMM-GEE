# Advanced Numerical Data Analysis: LMM and GEE

![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)
![R ≥ 4.0](https://img.shields.io/badge/R-≥%204.0-blue.svg)
![Quarto ≥ 1.4](https://img.shields.io/badge/Quarto-≥%201.4-orange.svg)
![IDE: Positron](https://img.shields.io/badge/IDE-Positron-blueviolet.svg)
![Workflow: REAAAPP](https://img.shields.io/badge/Workflow-REAAAPP-teal.svg)


Reproducible **Linear Mixed Effects Model (LMM)** and **Generalised Estimating Equations (GEE)** analysis of longitudinal HbA1c trajectories in Type 2 Diabetes Mellitus, following the **REAAAPP** workflow (GET503 Advanced Numerical Data Analysis, DrPH, Universiti Sains Malaysia).

---

<details>
<summary>📑 Table of Contents</summary>

- [Overview](#overview)
- [Contributors](#contributors)
- [Folder Structure](#folder-structure)
- [Installation](#installation)
- [Usage](#usage)
- [Data](#data)
- [List of Analyses](#list-of-analyses)
- [Results](#results)
- [Disclaimer](#disclaimer)
- [License](#license)
- [Version History](#version-history)

</details>

---

## Overview

This repository contains one combined group assignment submission for GET503 Advanced Numerical Data Analysis, covering two complementary longitudinal methods applied to the same T2DM dataset:

| # | Method | Estimand | Key Output |
|---|--------|----------|------------|
| 1 | Linear Mixed Effects Model (LMM) | Subject-specific | Individual HbA1c trajectories, random intercept and slope, ICC, R²m / R²c |
| 2 | Generalised Estimating Equations (GEE) | Population-averaged | Marginal HbA1c change, QIC-selected working correlation, robust sandwich SE |

Both analyses share the same dataset (20,000 T2DM patients × 3 time points = 60,000 observations), the same REAAAPP analytical workflow, and a unified Quarto document with colourblind-safe visualisations (Okabe and Ito, 2008).

---

## Contributors

| Name | Student ID | Role |
|------|------------|------|
| Dr. Siti Nurhafizah binti Saharudin | -- | Analysis and Documentation |
| Dr. Ahmad Zulfahmi bin Sha'ari | -- | Analysis and Documentation |
| Dr. Nazirul Munir bin Abu Hassan | 23202537 | Analysis and Documentation |
| Dr. Mohd Fittri Fahmi bin Fauzi | -- | Analysis and Documentation |

---

## Folder Structure

```
├── T2DM_LMM_GEE_Combined.qmd
├── T2DM_LMM_GEE_Combined.html
├── T2DM_LMM_GEE_Combined.css
├── t2dm_clean.csv
└── README.md
```

- `T2DM_LMM_GEE_Combined.qmd` is the fully reproducible Quarto source file containing both LMM and GEE analyses
- `T2DM_LMM_GEE_Combined.html` is the rendered self-contained HTML report
- `T2DM_LMM_GEE_Combined.css` is the custom stylesheet for the HTML output
- `t2dm_clean.csv` is the cleaned longitudinal T2DM dataset (20,000 patients, 3 time points, 60,000 rows)
- `README.md` is this overview file

---

## Installation

### Prerequisites

- R (≥ 4.0.0) and [Positron](https://positron.posit.co/) (recommended IDE)
- Quarto (≥ 1.4)

### Required R Packages

```r
install.packages("pacman")

pacman::p_load(
  # Data wrangling
  tidyverse, janitor,
  # Tables
  gtsummary, flextable, knitr, kableExtra,
  # EDA
  skimr, summarytools, DataExplorer, corrplot,
  # LMM
  lme4, lmerTest, sjPlot, performance, MuMIn, emmeans,
  # GEE
  geepack, broom,
  # Visualisation
  ggplot2, patchwork, scales, purrr,
  # Utilities
  here, pacman
)
```

### Setup

```bash
git clone https://github.com/DrKangMunir/advance-numerical-analysis.git
cd advance-numerical-analysis
```

Place `t2dm_clean.csv` in the same folder as `T2DM_LMM_GEE_Combined.qmd` before rendering.

---

## Usage

```bash
quarto render T2DM_LMM_GEE_Combined.qmd
```

Alternatively, open `T2DM_LMM_GEE_Combined.qmd` in **Positron** and click **Render**.

> **Data path:** If `t2dm_clean.csv` is not in the same folder as the QMD, update `data_path` in the `data-prep` chunk to the full absolute path before rendering.

---

## Data

### T2DM Longitudinal Treatment Outcomes

- **File:** `t2dm_clean.csv`
- **Source:** Rana, S. *T2DM Longitudinal Treatment Outcomes* [Dataset]. Kaggle, 2024. Available at: [https://www.kaggle.com/datasets/sahilrana034/t2dm-longitudinal-treatment-outcomes](https://www.kaggle.com/datasets/sahilrana034/t2dm-longitudinal-treatment-outcomes)
- **Sample size:** n = 20,000 patients × 3 time points = 60,000 observations
- **Design:** Balanced longitudinal panel; each patient measured at T1_Pre (baseline), T2_Mid (mid-intervention), and T3_Post (post-intervention)

| Variable | Type | Description |
|----------|------|-------------|
| `hba1c` | Continuous | HbA1c (%) — primary outcome; measure of glycaemic control |
| `patient_id` | Factor | Unique patient identifier; cluster variable for LMM and GEE |
| `time_step` | Ordered factor | Time point: T1_Pre (ref), T2_Mid, T3_Post |
| `time_num` | Numeric (0/1/2) | Numeric encoding of time for the random slope model |
| `intervention_strength` | Ordered factor | Intervention tier: Weak (ref), Standard, Intensive |
| `age` | Continuous | Age in years |
| `gender` | Factor | Female (ref), Male |
| `bmi` | Continuous | Body Mass Index (kg/m²) |
| `physical_activity_level` | Ordered factor | Low (ref), Moderate, High |
| `diet_score` | Continuous | Dietary quality score; higher = better |
| `sleep_hours` | Continuous | Average daily sleep (hours) |
| `medication_adherence` | Ordered factor | Poor (ref), Moderate, Good |
| `comorbidity_level` | Ordered factor | Low (ref), Medium, High |

> **Note:** This is a synthetic educational dataset. Results are not intended for clinical or policy application.

---

## List of Analyses

### Analysis 1: Linear Mixed Effects Model (LMM)

The LMM analysis follows the REAAAPP workflow from data loading through to a publication-quality model table and forest plot.

- **Data preparation** covers CSV loading, explicit factor-level encoding with clinically meaningful reference categories (Female, Weak intervention, Low activity, Poor adherence, Low comorbidity), and numeric time encoding required for the random slope extension
- **Exploratory data analysis** includes a spaghetti plot of 100 randomly sampled individual HbA1c trajectories with a group mean overlay, mean trajectory plots by intervention group, violin and box plots of HbA1c distribution by time and intervention, and a Pearson correlation matrix of all continuous predictors
- **Assumption checking** covers visual normality assessment (Q-Q plot and histogram) with justification for skipping Shapiro-Wilk at n = 60,000, and ICC estimation from the null model. ICC = 0.813 confirms that 81.3% of total variance is between-patient, strongly justifying LMM over standard regression
- **Model building** fits five sequential models: (1) null model for variance decomposition and ICC, (2) time-only unconditional growth model, (3) full fixed-effects model with all covariates, (4) random slope extension allowing each patient's rate of change to vary, and (5) Time x Intervention interaction model to test whether intervention groups diverge in HbA1c trajectories over time. All models use REML for parameter estimation; ML refitting is used only for fixed-effects comparison
- **Model comparison** uses likelihood ratio tests via `anova()` and AIC comparison across all five models to select the final model. The interaction model is confirmed as the best fit
- **Residual diagnostics** present residuals vs fitted and Q-Q plots for the final model confirming random scatter and approximate normality
- **Estimated marginal means** are derived using `emmeans` for within-group time contrasts and between-group pairwise contrasts at T3_Post with Bonferroni correction. At T3_Post: Intensive = 8.04%, Standard = 8.16%, Weak = 8.24%. All pairwise comparisons are significant
- **Model fit** reports marginal R² (R²m = 0.026) and conditional R² (R²c = 0.974), confirming that the random intercept absorbs 94.8% of total variance not explained by fixed effects
- **Final results** present a `sjPlot::tab_model()` table with all fixed-effect estimates, 95% CIs, random-effect variances, ICC, R², and AIC, alongside a forest plot from `sjPlot::plot_model()`

---

### Analysis 2: Generalised Estimating Equations (GEE)

The GEE analysis uses the same dataset and REAAAPP structure, complementing LMM with population-averaged estimates.

- **Theoretical framework** presents the marginal model specification with identity link, the four working correlation structures (Independence, Exchangeable, AR(1), Unstructured) with their mathematical definitions, the QIC criterion formula (Pan, 2001) for structure selection, and the robustness property of the sandwich variance estimator (Liang and Zeger, 1986)
- **Univariate GEE screening** fits separate simple GEE models (Exchangeable) for each predictor. Variables with p < 0.25 are retained. Gender (Male) is the only non-significant predictor but is retained as an a priori biological confounder. Crude coefficients are visualised in a coefficient plot distinguishing significant (filled) from non-significant (open) estimates
- **Multivariable GEE: four structures** fits the full adjusted model simultaneously under Independence, Exchangeable, AR(1), and Unstructured working correlation using `geepack::geeglm()` with robust sandwich standard errors. Summaries for all four structures are presented
- **QIC-based model selection** compares all four structures. Exchangeable achieves the lowest QIC (44,871.88) and is selected as the primary inference model. Working correlation parameters show alpha = 0.959 under Exchangeable and 0.958 under AR(1), confirming that HbA1c correlations across all time lags are nearly identical and that any pair-specific structure adds no explanatory value
- **Time x Intervention interaction** fits the interaction GEE model under AR(1) and derives estimated marginal means by time and intervention group. All interaction terms are significant (p < 0.0001). At T3_Post: Intensive = 7.96%, Standard = 8.11%, Weak = 8.26%
- **Model diagnostics** plot Pearson residuals vs fitted values and a Q-Q plot for the Exchangeable model. Random scatter around zero and an approximately straight Q-Q diagonal confirm adequate mean model specification. The observed mean trajectory plot validates that GEE-estimated EMMs closely track raw group means
- **Sensitivity analysis** presents key coefficients (time effects, intervention effects) across all four working correlation structures in a formatted table. Time effects and Intensive intervention are significant across all structures, confirming structural invariance of primary conclusions
- **Final results** present a publication-quality table of GEE Exchangeable estimates with predictor labels, beta coefficients, 95% robust CIs, and p-values, alongside a multivariable coefficient plot colour-coded by predictor group (time, intervention, demographics, lifestyle, clinical)
- **Results interpretation** summarises population-averaged findings: the Intensive group achieves a 0.30 percentage point lower HbA1c than Weak at T3_Post; lifestyle factors (physical activity, sleep, medication adherence, diet) dominate effect magnitudes, exceeding the intervention effects; Standard intervention is not statistically significant under the QIC-preferred Exchangeable structure
- **Public health applications** translate findings into three actionable recommendations: Intensive intervention prioritisation for Malaysian CPG T2DM revision, multi-component lifestyle modification targeting all four lifestyle domains simultaneously, and adoption of GEE for population-level glycaemic surveillance in the National Diabetes Registry Malaysia

---

## Results

### Linear Mixed Effects Model

- The null model ICC of **0.813** confirmed that 81.3% of total HbA1c variance is attributable to stable between-patient differences, validating the need for LMM over standard regression
- All four Time x Intervention interaction terms were highly significant (p < 0.0001)
- At T3_Post, the Intensive group achieved an adjusted mean HbA1c of **8.04%** versus **8.24%** for Weak (gap: 0.20 pp, p < 0.0001 after Bonferroni correction)
- The largest modifiable predictors were high physical activity (beta = -0.52%, p < 0.001), sleep hours (beta = -0.41%, p < 0.001), and good medication adherence (beta = -0.36%, p < 0.001)
- Conditional R² = 0.974, confirming excellent overall model fit

### Generalised Estimating Equations

- QIC selected **Exchangeable** as the optimal working correlation structure (QIC = 44,871.88 vs AR(1) = 52,069.55)
- Only **Intensive intervention** achieved a significant population-averaged reduction (beta = -0.077%, p = 0.005) after full covariate adjustment; Standard was non-significant (p = 0.259)
- Lifestyle factors dominated population-averaged effect magnitudes: Physical Activity High (beta = -0.516%), Sleep hours (beta = -0.406%), Medication Adherence Good (beta = -0.355%)
- The Time x Intervention interaction confirmed a **30-fold widening** of the Intensive vs Weak gap between T1_Pre and T3_Post at the population level
- Full results are available in `T2DM_LMM_GEE_Combined.html`

---

## Disclaimer

- This repository was developed as part of the **GET503 Advanced Numerical Data Analysis** course, DrPH Epidemiology Programme, Universiti Sains Malaysia.
- The dataset is a synthetic public-use dataset sourced from Kaggle and is used strictly for educational purposes. Beta coefficients and estimated marginal means reflect sample-level associations and should not be used for clinical or policy decisions without validation on real patient data.
- Code and documentation were developed with assistance from **Claude (Anthropic)**, an AI assistant. All analytical choices, model specifications, interpretations, and conclusions remain the sole responsibility of the authors.
- Materials are intended exclusively for **educational and demonstration purposes**.

---

## License

This project is licensed under the **MIT License**.

---

## Version History

- v1.0.0 (2026-06-10): Initial release with combined LMM and GEE analysis in a single Quarto document following the REAAAPP workflow.
