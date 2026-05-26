# [EM1413] Health Data Science
## Hospital Emigration in Italy — Gravity Model & Predictive Analysis

**Author:** Faccin Giacomo — Student ID: 892389  
**Date:** May 22, 2026  
**Language:** R (R Markdown → HTML)

---

## Overview

This report analyses **inter-regional hospital patient mobility in Italy**, modelling the phenomenon by which patients travel outside their region of residence to receive medical care. The study covers three clinical specialties — **Cardiology**, **Oncology**, and **Orthopaedics** — and compares three modelling approaches: a **GLMM Gravity Model** for inferential analysis, and **XGBoost** and **Random Forest** for predictive modelling.

---

## Data

Six datasets in Stata (`.dta`) format, sourced from ISTAT and the Italian Ministry of Health, organised by specialty and flow type (patient counts and monetary values in euros).

Each observation is an **origin region × destination region × year** triplet, with 31 variables covering:

- Patient characteristics (age, sex, length-of-stay, Elixhauser score)
- Regional supply/demand indicators (hospital beds, GDP, population)
- Geographic features (inter-regional distance)

Key preprocessing steps include normalisation of bed counts per 1,000 inhabitants, construction of destination/origin ratios for beds, GDP and quality, and listwise deletion of missing values arising from zero-population denominators.

---

## Report Structure

### Section 0 — Setup & Data Loading
Library setup, data loading, variable description, feature engineering, region label mapping, and first descriptive statistics. The datasets show strong right-skewness, with zero-flow shares ranging from 30.6% (Orthopaedics) to 43.5% (Cardiology).

### Section 1 — Exploratory Data Analysis (EDA)
- **1.1** Density plots of hospital migration flows — log(+1) transformation to handle skewness
- **1.2** Evolution of total flows over time — notable increase in Orthopaedics
- **1.3** South → North migration share — high proportions supporting healthcare inequality hypotheses
- **1.4** Origin-Destination heatmap — identifies attraction poles (Lombardy, Emilia-Romagna) and major origin regions (Campania, Calabria)
- **1.5** Correlation matrix — preliminary associations between flows, distance, GDP gap, quality gap, and bed availability

### Section 2 — Inferential Analysis: GLMM Gravity Model
The theoretical framework is the **gravity model**, where patient flows decrease with geographic distance and increase with the demographic and healthcare mass of origin and destination regions:

$$F_{ij} = \alpha \cdot \frac{M_i^{\beta_1} \cdot M_j^{\beta_2}}{D_{ij}^{\beta_3}} \cdot \exp(\mathbf{X}_{ij}\boldsymbol{\gamma})$$

Three model specifications are compared:

| Model | Description |
|-------|-------------|
| **M1** | OLS Baseline |
| **M2** | OLS with Fixed Effects for Origin Region |
| **M3** | GLMM with Random Effects *(main model)* |

The ICC of 63.5% from the null model justifies the random effects structure. M3 is selected based on AIC comparison and residual diagnostics. Random effects capture persistent regional propensities to send or attract patients beyond what observed covariates explain.

### Section 3 — XGBoost
Two XGBoost variants are estimated — **OLS loss (MSE)** and **Quantile loss (α = 0.75)** — using a stratified train-test split to preserve the proportion of zero and high-flow observations. Learning curves and feature importance rankings are reported and compared across specialties.

### Section 4 — Random Forest
A Random Forest model is implemented as a machine learning benchmark, using the same train/validation/test sets as XGBoost. A baseline model with 500 trees is estimated and tuned via hyperparameter optimisation. Performance is evaluated using RMSE, MAE, and R², and feature importance is compared with XGBoost.

### Section 5 — Real World Model Comparison
All three models are re-estimated on the training set and compared on the same test set. Evaluation is stratified by flow volume (low, medium, high) to avoid masking poor performance on high-volume corridors — which are the most operationally relevant. Metrics: RMSE, MAE, Bias, Mean Predicted vs Mean Observed.

---

## Key Findings

- **GLMM** confirms the gravity model structure and is the most interpretable model. Preferred for policy interpretation.
- **XGBoost OLS** achieves the lowest RMSE and MAE on the test set. Best for operational prediction of patient flows.
- **Random Forest** provides consistent feature importance rankings across models, validating the stability of the machine learning results.

---

## Tools

- **Language:** R  
- **Key packages:** `lme4`, `xgboost`, `randomForest`, `ggplot2`, `kableExtra`, `dplyr`  
- **Report format:** R Markdown knitted to HTML via Pandoc
