# Predicting 30-Day Hospital Readmission Among Patients With Diabetes

**HSE 751: Programming for Health Data Science** · Final Project (Week 4 Checkpoint)

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/audreybyrne03/HSE751-Final-Project/blob/main/Diabetes_Readmission_Pipeline.ipynb)

## Project Overview

This repository contains a reproducible, end-to-end analysis pipeline that predicts **30-day hospital readmission** among patients with diabetes, using the UCI *Diabetes 130-US Hospitals for Years 1999–2008* dataset.

The pipeline covers:

- data import and validation
- cohort definition
- preprocessing and feature engineering
- descriptive and exploratory analysis
- inferential statistics
- five supervised machine learning models, with evaluation and comparison

It is also designed as a first implementation of a reusable **clinical outcomes analytics pipeline**. Dataset-specific definitions are kept in configuration dictionaries, separate from the reusable analysis functions, so the workflow can be adapted to other patient-level clinical datasets.

## Repository Contents

```
HSE751-Final-Project/
│
├── Diabetes_Readmission_Pipeline.ipynb                  # Main project notebook (current version)
├── Dataset_Selection_and_Final_Project_Planning_Lab.ipynb  # Week 3 planning lab (earlier version)
├── diabetic_data.csv                                    # Project dataset (UCI ID 296)
├── requirements.txt                                     # Python packages required
└── README.md
```

## Dataset

| | |
|---|---|
| **Name** | Diabetes 130-US Hospitals for Years 1999–2008 |
| **Source** | UCI Machine Learning Repository, dataset ID 296 |
| **URL** | https://archive.ics.uci.edu/dataset/296/diabetes+130-us+hospitals+for+years+1999-2008 |
| **File** | `diabetic_data.csv`, the original file distributed by UCI |
| **Size** | 101,766 hospital encounters from 71,518 patients; 50 columns |

The file includes two identifier columns, `encounter_id` and `patient_nbr`, that are not included in the `ucimlrepo` Python package version of the dataset. `patient_nbr` is used to split the data by patient, so that no patient appears in both the training and test sets.

In the raw file, missing values are coded as `?`. In the two laboratory columns (`max_glu_serum`, `A1Cresult`), `None` means the test was not performed, so it is kept as its own category rather than treated as missing.

## Prediction Problem

- **Outcome:** `readmitted_30d`. This is coded **1** if the encounter was followed by readmission within 30 days (`<30`) and **0** otherwise (`>30` or `NO`).
- **Analytic cohort:** 99,340 encounters from 69,987 patients. This excludes 2,423 encounters ending in death or hospice discharge, after which a readmission is not possible, and 3 encounters with an invalid gender value.
- **Outcome rate:** 11.4% (11,314 encounters). The outcome is imbalanced.
- **Primary metric:** **F1 score**. It balances precision and recall for the minority class, whereas accuracy would be misleading with an 11% positive rate.
- **Secondary metrics:** precision, recall, specificity, accuracy, AUROC, and PR-AUC.

## Analysis Pipeline

| Notebook section | Step |
|---|---|
| 1 | Dataset selection and justification |
| 2 | Data import from GitHub and automated validation against a data contract |
| 3–8 | Dataset structure, target definition, descriptive statistics, exploratory visualizations, and data quality assessment |
| 9–10 | Preprocessing plan and project planning (from Week 3) |
| 11 | Cleaning and preprocessing: cohort definition, binary target, removal of low-information predictors, and feature engineering |
| 12 | Inferential statistics: chi-square tests, Mann–Whitney U tests, effect sizes, Holm correction, and a sensitivity analysis |
| 13 | Machine learning: patient-level split, preprocessing pipeline, tuning, five models, threshold optimization, and evaluation |
| 14 | Interpretation of findings and limitations |
| 15 | Summary of pipeline enhancements |

## Key Results

**Inferential statistics:** 30-day readmission rose with the number of prior inpatient admissions, from **8.6%** with none to **26.4%** with three or more (χ²(3) = 2411, p < 0.001, Cramér's V = 0.16). The association held when each patient was counted only once. Across all predictors, only prior utilization and discharge disposition had more than a negligible association with readmission.

**Machine learning (held-out test set, patient-level split):**

| Model | Test F1 (95% CI) | Test AUROC (95% CI) |
|---|---|---|
| **XGBoost** | **0.290 (0.268–0.312)** | **0.681 (0.664–0.697)** |
| Random Forest | 0.287 (0.263–0.313) | 0.679 (0.661–0.695) |
| Decision Tree | 0.278 (0.256–0.301) | 0.653 (0.637–0.671) |
| Logistic Regression | 0.276 (0.259–0.293) | 0.665 (0.649–0.679) |
| Neural Network (MLP) | 0.272 (0.255–0.289) | 0.663 (0.648–0.678) |
| *Reference: predict all readmitted* | *0.203* | *0.500* |

XGBoost performed best. At its tuned threshold, it identified about 43% of readmissions with a precision almost twice the base rate. The confidence intervals overlap for all five models, so their differences are small. Discharge disposition and prior inpatient admissions were the most important predictors. Model performance is modest, which is consistent with published readmission models built from administrative data.

## Pipeline Enhancements

| # | Enhancement | Section |
|---|---|---|
| 1 | Data contract and automated validation, with loading from GitHub and clear error messages | 2 |
| 2 | Rule-based cohort definition with an automated cohort flow table | 11.2 |
| 3 | Automated removal of low-information predictors using configurable rules | 11.4 |
| 4 | Clinical feature engineering: ICD-9 diagnosis grouping, numeric age, total prior visits, medication changes | 11.5 |
| 5 | Automated statistical testing framework with effect sizes and Holm correction | 12 |
| 6 | Decision-threshold optimization and expanded evaluation metrics | 13.5 |
| 7 | Model registry with a modular training, tuning, and comparison workflow | 13 |
| 8 | Patient-level bootstrap confidence intervals for test performance | 13.7 |

Each enhancement is documented in the notebook with what was modified, why it was implemented, and how it improves the workflow.

## How to Reproduce

1. Click the **Open in Colab** badge at the top of this page.
2. Select **Runtime → Run all**.

The notebook loads `diabetic_data.csv` directly from this repository, so no file upload is needed. All required packages are pre-installed in Google Colab. A full run takes about **3–5 minutes**.

To run the notebook locally instead:

```bash
git clone https://github.com/audreybyrne03/HSE751-Final-Project.git
cd HSE751-Final-Project
pip install -r requirements.txt
jupyter notebook Diabetes_Readmission_Pipeline.ipynb
```

When `diabetic_data.csv` is in the same folder as the notebook, the local copy is used automatically. All random processes use a fixed seed (`RANDOM_STATE = 42`), so results are reproducible.

## Software

Python 3 with: `pandas`, `numpy`, `matplotlib`, `seaborn`, `scipy`, `statsmodels`, `scikit-learn`, and `xgboost` (see `requirements.txt`).

## Reference

Strack B, DeShazo JP, Gennings C, et al. Impact of HbA1c measurement on hospital readmission rates: analysis of 70,000 clinical database patient records. *BioMed Research International.* 2014;2014:781670.
