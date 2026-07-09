# multi-cohort-breast-cancer-biomarker-discovery
A machine learning pipeline for breast cancer diagnosis and biomarker discovery, validated across three independent cohorts (Wisconsin, METABRIC, GSE25066). Combines Random Forest classification, SHAP-based explainability, statistical validation, and survival analysis to identify a cross-platform reproducible biomarker panel.
# Explainable ML for Multi-Cohort Breast Cancer Biomarker Discovery

Explainable machine learning classifiers built and cross-validated across three independent breast cancer cohorts — one morphological (Wisconsin) and two transcriptomic (METABRIC, GSE25066) — to identify biomarkers that generalize across datasets and measurement platforms, rather than being artifacts of a single cohort.

## Overview

A recurring concern in ML-for-biology work is that models and "biomarkers" discovered on one dataset don't generalize to new patient cohorts or platforms. This project addresses that by:

- Training classifiers on three independent cohorts of increasing complexity
- Using SHAP to extract biologically interpretable feature/gene rankings
- Statistically validating SHAP-ranked genes (Mann–Whitney U + FDR correction)
- Testing bootstrap stability of the biomarker ranking across 30 resampling runs
- Externally validating a METABRIC-trained model on an independent, cross-platform cohort (GSE25066)
- Linking top biomarkers to patient survival via Kaplan–Meier and Cox Proportional Hazards analysis

## Datasets

| Dataset | Source | Samples | Features | Task |
|---|---|---|---|---|
| Wisconsin Diagnostic Breast Cancer | UCI ML Repository / Kaggle | 569 | 30 morphological features | Benign vs. malignant classification |
| METABRIC | cBioPortal-format export | 1,904 | 489 gene-expression + clinical/mutation columns | ER status & PAM50 molecular subtype classification |
| GSE25066 (external) | NCBI GEO | 508 (182–502 after filtering) | 22,283 probes → 431 common genes | External validation of ER-status model |

## Methodology

```
Problem Definition → Data Collection → Data Cleaning → EDA → Feature Engineering
→ Model Selection → Model Training → Hyperparameter Tuning → Evaluation
→ Prediction → Biological Interpretation
```

**Preprocessing:** median imputation (GSE25066), categorical encoding (diagnosis, ER status, PAM50 subtype), `StandardScaler` fit on training data only, and cross-platform harmonization by mapping GSE25066 Affymetrix probe IDs to gene symbols (GPL96 annotation) then intersecting with METABRIC genes (431 common genes).

**Models trained (Wisconsin):** Logistic Regression, K-Nearest Neighbors, Decision Tree, Random Forest, SVM (RBF kernel). Random Forest was carried forward as the primary model for METABRIC and GSE25066 due to strong performance, class-imbalance handling, and SHAP `TreeExplainer` compatibility.

**Validation:** 80/20 stratified train-test split, 5-fold Stratified K-Fold cross-validation, `GridSearchCV` hyperparameter tuning on Random Forest.

## Results

### Wisconsin — Morphological Classification

| Model | Accuracy | AUC-ROC | F1 Score | CV AUC (mean ± sd) |
|---|---|---|---|---|
| Random Forest | 97.37% | 0.9965 | 0.9630 | 0.9865 ± 0.0102 |
| Logistic Regression | 96.49% | 0.9960 | 0.9512 | 0.9958 ± 0.0047 |
| Support Vector Machine | 97.37% | 0.9947 | 0.9630 | 0.9948 ± 0.0046 |
| K-Nearest Neighbors | 95.61% | 0.9823 | 0.9383 | 0.9871 ± 0.0137 |
| Decision Tree | 92.11% | 0.9448 | 0.8861 | 0.8875 ± 0.0607 |

After `GridSearchCV` tuning, the Random Forest reached **97.37% accuracy** and **0.9974 AUC-ROC** (best params: `max_depth=5`, `min_samples_leaf=1`, `n_estimators=100`).

Top morphological biomarkers by mean |SHAP value|: `concave points_worst`, `area_worst`, `perimeter_worst`, `concave points_mean`, `radius_worst`.

### METABRIC — Gene Expression Classification

| Task | Accuracy | AUC-ROC |
|---|---|---|
| ER+ vs. ER- (binary) | 94.75% | 0.9859 |
| PAM50 subtype (6-class) | 77.37% | — |

Top gene biomarkers by mean |SHAP value|: **GATA3**, **MAPT**, **IGF1R**, **BCL2**, **EGFR**, **CDK6**, **BMPR1B**, **HSD17B4**, **CCNE1**, **TBX3** — all significant at FDR-corrected p < 0.001 (Mann–Whitney U).

### GSE25066 — External Validation

| Cohort | Role | Accuracy | AUC-ROC |
|---|---|---|---|
| METABRIC (shared-gene subset) | Internal | 94.75% | 0.9863 |
| GSE25066 | External | 68.51% | 0.8502 |

An AUC drop of 0.1361 reflects platform/population differences rather than model failure. **17 of the top 20 SHAP-ranked genes overlapped** between METABRIC and GSE25066 (85% overlap), supporting genuine biological signal.

### Core Validated Biomarker Panel

| Rank | Gene | Found In | Known Role |
|---|---|---|---|
| 1 | GATA3 | METABRIC, GSE25066 | Master TF for luminal/ER+ identity |
| 2 | BCL2 | METABRIC, GSE25066 | Anti-apoptotic, ER-driven |
| 3 | CDK6 | METABRIC, GSE25066 | Cell-cycle regulator (G1/S) |
| 4 | CHEK1 | METABRIC, GSE25066 | DNA-damage checkpoint kinase |
| 5 | BMPR1B | METABRIC, GSE25066 | BMP signalling, luminal subtype |

Stability confirmed via 30-run bootstrap resampling (10 genes in 100% of runs). Cox Proportional Hazards modeling on these 5 genes showed no individual gene reached significance (p < 0.05) as an independent prognostic factor for overall survival.

## Tools & Libraries

- **Language:** Python 3
- **Core:** NumPy, Pandas, Scikit-learn
- **Explainability:** SHAP
- **Statistics:** SciPy (Mann–Whitney U), statsmodels / lifelines (Kaplan–Meier, Cox PH, log-rank)
- **Visualization:** Matplotlib, Seaborn
- **Environment:** Jupyter Notebook, Google Colab
- **Bioinformatics:** GPL96 platform annotation (probe-to-gene mapping), cBioPortal-format METABRIC data, NCBI GEO series matrix (GSE25066)

## Repository Structure

```
outputs/
  week1/   # data acquisition & preprocessing outputs
  week2/   # EDA & Wisconsin model results
  week3/   # METABRIC model results
  week4/   # GSE25066 external validation, survival analysis, final report/poster
```

## Key Findings

- Random Forest consistently gave the best or joint-best AUC-ROC across all three datasets/tasks.
- A biologically consistent, statistically validated, cross-platform biomarker panel (GATA3, BCL2, CDK6, CHEK1, BMPR1B) was identified.
- External, cross-platform validation showed meaningful generalization (AUC 0.85) despite a modest, expected performance drop from internal validation.

## Future Scope

- Extend with deep-learning / multi-omics models (mutations, copy-number, methylation)
- Validate on additional independent cohorts
- Combine with treatment-response data to move toward predictive biomarkers of therapy response

## References

- Pedregosa, F. et al. *Scikit-learn: Machine Learning in Python.* JMLR (2011)
- Lundberg, S. M. & Lee, S.-I. *A Unified Approach to Interpreting Model Predictions (SHAP).* NeurIPS (2017)
- Davidson-Pilon, C. *lifelines: survival analysis in Python*
- Wisconsin Diagnostic Breast Cancer Dataset — UCI ML Repository / Kaggle
- METABRIC — accessed via [cBioPortal](https://www.cbioportal.org)
- GSE25066 — [NCBI GEO](https://www.ncbi.nlm.nih.gov/geo), incl. GPL96 platform annotation

## Author

**Divya S** — Stella Maris College
Internship: AI & Machine Learning in Computational Biology, Biotechtrek
