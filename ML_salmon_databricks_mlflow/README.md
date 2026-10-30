# Predicting Next-Week Adult Female Salmon Lice Levels

A machine-learning workflow for one-week-ahead forecasting of adult female salmon lice levels at Norwegian aquaculture localities, using weekly public data from BarentsWatch (2012–2026).

## Project Objective

This project develops and evaluates a supervised machine-learning model for predicting next-week adult female salmon lice levels at Norwegian aquaculture localities.

The main modelling question is:
> Can a supervised machine-learning model improve one-week-ahead predictions compared with a simple persistence baseline that assumes next week's lice level will be the same as the current week's level?

The analysis focuses on active salmonid localities with valid weekly lice reports and compares several regression models using a time-based train-validation-test design.

## Data Source

Weekly locality-level fish-health data were obtained from the [BarentsWatch Fish Health API](https://www.barentswatch.no/) for 2012–2026.

The raw data include locality information, salmon lice measurements, sea temperature, treatment information and locality status.

Main variables used in the modelling workflow:
- `adult_female_lice`: current-week adult female lice level
- `mobile_lice`: current-week mobile lice level
- `stationary_lice`: current-week stationary lice level
- `sea_temperature`: reported sea temperature
- `n_treatments`: number of registered treatments
- `week`: ISO week of year

The prediction target is the adult female lice level at the same locality in the following calendar week.

## Processing Logic

Raw JSON → Combine weekly records → Standardise column names → Filter modelling population → Data-quality checks → Construct calendar week date → Construct next-week target

### Modelling Population
The modelling dataset was restricted to locality-weeks that:
1. had a salmonid aquaculture licence,
2. were not classified as fallow,
3. had a valid lice report.

This avoids treating structurally missing lice measurements as zero and ensures that the model is trained only on weeks with valid observed lice levels.

### Sea-temperature Quality Control
A small number of sea-temperature observations contained implausibly high values. Because the underlying error mechanism could not be established reliably, these values were not manually corrected. Instead, temperatures above 25°C were treated as missing for modelling.

### Next-week Target
The data contain gaps in reporting and locality activity. Therefore, simply shifting each locality's observations by one row could incorrectly pair observations that were two or more weeks apart. ISO calendar dates were created for each locality-week, and only observations whose next valid record occurred exactly seven days later were retained.

### Features and Target
The initial feature set contained current-week information only: adult female lice, mobile lice, stationary lice, sea temperature, number of treatments, and week of year. `total_lice` was excluded.

### Time-based Data Split
Because this is a time-series forecasting problem, random train-test splitting was avoided. The data were separated:
- **Training:** 2012–2022
- **Validation:** 2023–2024
- **Test:** 2025
- **Out-of-time evaluation:** available weeks of 2026

## Results

| Dataset | Model | MAE | RMSE |
| :--- | :--- | ---: | ---: |
| 2025 Test | Baseline | 0.09751 | 0.25246 |
| 2025 Test | GB | **0.09293** | **0.22391** |
| 2026 OOT | Baseline | **0.05439** | 0.12709 |
| 2026 OOT | GB | 0.05443 | **0.10979** |

On the 2025 test set, the gradient boosting model improves both MAE and RMSE over the persistence baseline. On the 2026 out-of-time data, RMSE improves substantially while MAE is essentially on par with the baseline. The difference in absolute error levels between 2025 and 2026 is likely driven by differences in the data distribution (2025 contains more extreme and less concentrated lice values).

## How to Run

### Local Environment
Requires Python 3.9+ and the following packages:
```bash
pip install pandas numpy requests scikit-learn matplotlib seaborn python-dotenv pyarrow