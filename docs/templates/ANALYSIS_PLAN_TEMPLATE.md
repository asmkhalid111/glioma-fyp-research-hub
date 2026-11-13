# Pre-Registered Analysis Plan Template

> **Template Status:** Official Research Template  
> **Applicable Milestone:** Milestone M1 (Pre-registration before model training)  
> **Guideline Compliance:** TRIPOD+AI Statement

---

## 1. Study Title & Investigators
- **Title:** [Proposed Study Title]
- **Authors/Investigators:** [Team Members & Roles]
- **Date of Pre-Registration:** [Date]
- **Version:** 1.0 (Locked)

---

## 2. Research Questions & Hypotheses
- **Primary Research Question:** [State primary question]
- **Secondary Research Questions:** [State secondary questions]
- **Pre-Specified Hypotheses:** [State directional hypotheses]

---

## 3. Dataset & Cohort Definitions
- **Training & Internal CV Cohort:** [TCGA-LGG / TCGA-GBM criteria, sample filters]
- **External Validation Cohort:** [CGGA criteria, sample filters, exclusion of recurrent/secondary]
- **Inclusion Criteria:** [Exact criteria]
- **Exclusion Criteria:** [Exact criteria]
- **Target Label Definitions:**
  - Baseline Label: Historical WHO 2007/2016 Grade (II, III, IV)
  - Primary Label: WHO CNS5-approximate molecular grades

---

## 4. Cross-Validation & Leakage Control
- **Partitioning Scheme:** [e.g., 5x5 Nested Stratified Grouped CV]
- **Grouping Variable:** Patient Barcode (`bcr_patient_barcode`)
- **In-Fold Transformations:** [List exact in-fold scaling, filtering, and feature selection]
- **Imbalance Handling:** [In-fold class-weighted loss / SMOTE]

---

## 5. Candidate Model Architectures & Baselines
- **Primary Classifiers:** [Elastic-Net Logistic Regression, Random Forest, XGBoost, Support Vector Machine]
- **Clinical/Demographic Baseline:** [Age + Karnofsky / Clinical covariates]
- **Hyperparameter Tuning:** [Optuna inner-CV search space and budget]

---

## 6. Evaluation Metrics & Statistical Analysis
- **Primary Metrics:** Balanced Accuracy, Macro-F1, Matthews Correlation Coefficient (MCC)
- **Secondary Metrics:** Per-Class Recall, Specificity, AUROC, AUPRC
- **Uncertainty Quantification:** 95% Bootstrap Confidence Intervals (B >= 1,000)
- **External Validation Protocol:** [Locked model application on CGGA with no threshold tuning]

