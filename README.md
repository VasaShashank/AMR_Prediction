# Predictive AMR Research Pipeline

Research pipeline for estimating antimicrobial resistance (AMR) risk from
structured clinical and microbiology features available before susceptibility
results are known. The project includes a reproducible notebook and aggregate
analysis artifacts for model comparison, feature selection, calibration, and
threshold analysis.

> **Research and data-use notice**
>
> This repository is for research and reproducibility. Use any restricted
> source data only through its approved access process and applicable data-use
> terms. Do not add source records, patient identifiers, row-level predictions,
> or other restricted data to this repository. The checked-in reports are
> aggregate summaries intended for analysis and reporting.
>
> The model is not a medical device and must not be used for diagnosis,
> treatment selection, or autonomous clinical decisions. The reported results
> require independent validation before any prospective use.

## Overview

The pipeline evaluates pre-susceptibility AMR prediction using a patient-grouped
holdout design. It combines feature reduction, model benchmarking, probability
calibration, confidence intervals, and clinically interpretable threshold
operating points.

The current output contains **355,331 rows and 136 columns** in the analyzed
feature table. The train/test partition contains **11,412 training patients**
and **2,853 test patients**, with **zero patient overlap**.

## Workflow

1. Prepare structured clinical, microbiology, and prior-exposure features.
2. Remove redundant or highly correlated predictors.
3. Rank candidate predictors and evaluate feature subsets from 10 to 30 inputs.
4. Select the parsimonious feature count using joint AUROC and PR-AUC
   distance-to-ideal optimization.
5. Compare linear, bagging, boosting, and ensemble models.
6. Calibrate probabilities and evaluate discrimination, calibration, and
   threshold-dependent performance on the held-out test partition.

## Results

### Feature selection

The search evaluated 95 SHAP-ranked candidate features. The selected solution
uses **16 features**, achieving a feature-search PR-AUC of **0.5902** and
AUROC of **0.8547**. The selected predictors are:

`ab_name`, `prior_resistant_count`, `org_name`, `sex_M`, `age`,
`spec_type_desc`, `age_x_comorbidities`, `exp_fluoroquinolone`, `icu_admission`,
`hemoglobin_last_value`, `test_seq`, `admission_location`,
`exp_cephalosporin`, `antibiotic_count_30d`, `icu_los_prior`, and
`days_since_last_antibiotic`.

### Model comparison

Test-partition metrics from `model_comparison_metrics.csv` and
`auroc_prauc_ci_95.csv`:

| Model | AUROC | PR-AUC | Brier | ECE |
| --- | ---: | ---: | ---: | ---: |
| Soft Voting Ensemble | 0.8553 (0.8520-0.8588) | 0.5881 (0.5795-0.5967) | 0.1549 | 0.2010 |
| XGBoost | 0.8549 (0.8516-0.8583) | 0.5889 (0.5805-0.5975) | 0.1529 | 0.1933 |
| LightGBM | 0.8549 (0.8517-0.8585) | 0.5863 (0.5777-0.5949) | 0.1555 | 0.1997 |
| CatBoost | 0.8497 (0.8463-0.8532) | 0.5744 (0.5655-0.5835) | 0.1596 | 0.2099 |
| Random Forest | 0.7694 (0.7652-0.7739) | 0.4376 (0.4288-0.4456) | 0.1961 | 0.2605 |
| Logistic Regression | 0.7057 (0.7004-0.7107) | 0.3403 (0.3325-0.3483) | 0.2182 | 0.3168 |

The final calibrated XGBoost report records AUROC **0.8526**, PR-AUC **0.5732**,
Brier score **0.1033**, and ECE **0.0044** at the primary threshold of **0.16**.

### Calibration

| Method | Brier score | ECE |
| --- | ---: | ---: |
| Uncalibrated | 0.1518 | 0.1917 |
| Platt (sigmoid) | 0.1021 | 0.0179 |
| Isotonic | 0.1007 | approximately 0.0000 |

### Threshold operating points

Thresholds should be selected for the intended evaluation objective rather than
treated as universal clinical rules.

| Operating point | Threshold | Sensitivity | Specificity | PPV | NPV |
| --- | ---: | ---: | ---: | ---: | ---: |
| Youden J | 0.16 | 79.79% | 74.17% | 39.53% | 94.55% |
| Maximum F1 | 0.27 | 62.69% | 86.73% | 49.98% | 91.66% |
| Sensitivity >= 70% | 0.21 | 70.98% | 81.43% | 44.71% | 92.99% |
| Sensitivity >= 90% | 0.10 | 90.27% | 59.12% | 31.84% | 96.63% |
| Specificity >= 90% | 0.30 | 54.58% | 90.59% | 55.10% | 90.41% |

## Repository layout

```text
AMR/
├── AMR_Prediction_Pipeline.ipynb
├── README.md
└── new_amr_output/
    ├── cache/
    ├── eda_plots/
    ├── manuscript_figures/
    ├── models/
    └── reports/
        ├── final_model_performance.csv
        ├── model_comparison_metrics.csv
        ├── auroc_prauc_ci_95.csv
        ├── calibration_comparison.csv
        ├── multi_threshold_performance.csv
        ├── selected_features.csv
        ├── feature_selection_summary.txt
        └── ...
```

The `reports/` directory contains aggregate evaluation tables, feature
lineage and selection summaries, subgroup analyses, calibration results, and
threshold interpretations. The `models/` directory contains generated model
artifacts when available; treat them as research artifacts and do not expose
them as a clinical service without the required governance and validation.

## Reproduction

1. Obtain approved access to the source data independently.
2. Open `AMR_Prediction_Pipeline.ipynb` in Jupyter or VS Code.
3. Configure the local input paths and Python environment used by the notebook.
4. Run the notebook from data preparation through report generation.
5. Review the generated files under `new_amr_output/`.

The notebook and aggregate reports are intended to make the analysis traceable;
exact results can vary with software versions, preprocessing inputs, and
random seeds.

## Limitations

- Feature selection is data-driven and may capture associations that are not
  causal, actionable, or transportable.
- The reported evaluation is internal held-out testing, not external or
  temporal validation.
- Performance metrics are sensitive to prevalence, cohort construction,
  missingness, and threshold choice.
- Clinical deployment would require expert review, leakage assessment,
  prospective availability checks, subgroup and fairness analysis, external
  validation, calibration monitoring, and clinical-impact evaluation.
