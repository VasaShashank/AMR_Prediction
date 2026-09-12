# Predictive AMR Research Pipeline

> **Privacy and clinical-use notice:** This repository is intended to share the
> reproducible analysis notebook and project description only.
> Do not add, upload, or commit PhysioNet/MIMIC-IV data or row-level outputs.

This is a research-oriented machine-learning pipeline for antimicrobial
resistance (AMR) prediction using the MIMIC-IV microbiology dataset. The
dataset must be obtained and used separately under the applicable PhysioNet
data-use agreement. No patient-level data is included in this repository.

## Important methodological limitation

The features in this version were selected **data-first**, rather than through
established clinical or microbiology domain knowledge. The pipeline searched
for feature subsets that maximized predictive objectives such as accuracy,
area under the precision-recall curve (AUPRC), and area under the ROC curve
(AUROC), with additional model and calibration metrics.

That optimization target may select associations that do not represent causal,
clinically actionable, or transportable biology. Consequently, the resulting
feature set and reported performance must not be interpreted as a validated
clinical decision rule. Real-world medical projects should combine domain
expert review, prospective availability checks, leakage assessment, fairness
and subgroup analysis, external/temporal validation, calibration, and
prospective clinical-impact evaluation before any clinical use.

This project is not medical advice and is not approved for diagnosis,
treatment selection, or autonomous clinical decisions.

This repository contains the clinical machine learning pipeline real-time, pre-susceptibility antimicrobial resistance (AMR) risk assessment in the ICU. The framework is trained on the MIMIC-IV electronic health records database.

## Study Overview

empiric antibiotic therapy in the ICU must begin 48--72 hours before susceptibility test results are completed. Delayed active therapy leads to high septic mortality, whereas blanket administration of broad-spectrum antibiotics accelerates the emergence of multi-drug resistant organisms. 

This framework resolves this dilemma by predicting patient-specific AMR risk at the precise moment a microbiology culture is ordered. The model achieves clinical-grade predictive discrimination and probability calibration using a parsimonious set of **16 feature inputs** selected through a data-driven progressive feature reduction search.

### Clinical Generalizability & Identifier Policy
In contrast to prior versions of this model, all database-specific clinical codes (such as MIMIC-IV's `ab_itemid`, `org_itemid`, `spec_itemid`, and `test_itemid`) are **dropped** during preprocessing. Validation sweeps verified that using text-based clinical names (e.g. `ab_name`, `org_name`, `spec_type_desc`) results in negligible performance changes ($\Delta$AUROC < 0.002) while eliminating dependency on site-specific database dictionary schemas. This allows direct deployment across different hospitals without schema matching.

---

## Technical Stack & Performance Summary

### 1. Machine Learning Pipeline (`Untitled copy.py`)
- **Cohort Size**: 355,331 culture-antibiotic pairs
- **Train/Test Strategy**: Patient-grouped split (80% Train, 20% Held-out Test) with 3-fold Stratified Group K-Fold tuning. Zero patient overlap between partitions.
- **De-duplication**: 32 collinear features ($r > 0.90$) removed.
- **Parsimony Search**: Dual-metric Distance-to-Ideal optimization selected an optimal **22-feature** subset.
- **Tuning**: Optuna hyperparameter optimization.
- **Model Leaderboard (Test Partition)**:
  - **Soft Voting Ensemble**: AUROC: **0.8560** (95% CI: 0.8529--0.8594) | PR-AUC: **0.5895** (95% CI: 0.5810--0.5981)
  - **XGBoost (calibrated, deployed)**: AUROC: **0.8514** | PR-AUC: **0.5716** | Brier Score: **0.1035** | ECE: **0.0049**
  - **LightGBM**: AUROC: **0.8548** (95% CI: 0.8515--0.8581) | PR-AUC: **0.5852** (95% CI: 0.5764--0.5937)
  - **CatBoost**: AUROC: **0.8514** (95% CI: 0.8482--0.8549) | PR-AUC: **0.5805** (95% CI: 0.5721--0.5893)
- **Calibration**: Isotonic regression (Brier: 0.1035, ECE: 0.0049).
- **Decision Thresholds**:
  - **Youden J (primary)**: Threshold: **0.17** | Sensitivity: **78.08%** | Specificity: **75.80%** | PPV: **40.57%** | NPV: **94.23%**
  - **Sensitivity >= 90%**: Threshold: **0.10** | Sensitivity: **89.88%** | Specificity: **59.51%**
