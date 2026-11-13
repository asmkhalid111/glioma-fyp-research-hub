# CURRENT_STATE.md — Current Research State & Technical Inventory

> **Document Status:** Active Baseline  
> **Last Updated:** 13 November 2026  
> **Source Documents:** [incoming/01_research_project_understanding.md](incoming/01_research_project_understanding.md), [incoming/02_independent_scientific_audit.md](incoming/02_independent_scientific_audit.md)  
> **Epistemic Traceability:** All entries derived strictly from historical project understanding and scientific audit reports.

---

## 1. Explicit Knowledge Boundaries & Epistemic Separation

To ensure absolute scientific integrity, the research state is partitioned into five distinct epistemic categories (`02_audit` §0):

### 1.1 Established by Original Research Documents
- The original research problem addressed three-class grading of adult diffuse gliomas (Grades II, III, IV) using bulk RNA-seq (`star_tpm`) from `TCGA-LGG` and `TCGA-GBM`.
- Histological Grade II vs III discrimination is subjective with documented interobserver discordance.
- WHO CNS5 (2021) established that grading must be conducted within molecularly defined types (IDH-mutant astrocytoma, 1p/19q-codeleted oligodendroglioma, IDH-wildtype glioblastoma).
- The existing master dataset comprises 708 rows (Grade II: 320, Grade III: 213, Grade IV: 175).
- Grade IV samples originate 100% from TCGA-GBM (predominantly IDH-wildtype); Grades II and III originate 100% from TCGA-LGG (predominantly IDH-mutant). Grade IV prediction is therefore confounded by cohort and molecular differences.
- Eighteen specific methodological weaknesses (W1–W18) were identified by the audit.

### 1.2 Directly Inspected by Antigravity in Workspace
- Foundational research records: [incoming/01_research_project_understanding.md](incoming/01_research_project_understanding.md) and [incoming/02_independent_scientific_audit.md](incoming/02_independent_scientific_audit.md) (both verified via SHA-256).
- Archival and documentation copies in `docs/historical/`, `docs/01_discovery/`, and `docs/02_scientific_audit/`.
- Local Git repository status: Initialized on `main`, synchronized with private GitHub repository `https://github.com/asmkhalid111/glioma-fyp-research-hub`.
- Multi-model collaboration framework: [AI_CONTEXT_PACK.md](AI_CONTEXT_PACK.md), [MULTI_MODEL_REVIEW_PROTOCOL.md](MULTI_MODEL_REVIEW_PROTOCOL.md), [MULTI_MODEL_REVIEW_PROMPT.md](MULTI_MODEL_REVIEW_PROMPT.md), and [docs/ai_reviews/](docs/ai_reviews/README.md).
- Absence of raw data files, patient-level records, or notebooks in the local workspace.

### 1.3 Reported But Unreproduced
- Historical Colab test metrics: SVM 80.99%, RF 78.17%, tuned RF 77.46%, Grade II/III subset accuracy 62%–71%, Grade IV recall ~0.97.
- Phase-I report headline claim: **84.51%** Random Forest accuracy with log2(TPM+1), MI+LASSO+RF, top 500 genes on an 80/10/10 split. (Code shows no log transform, variance filter to 5,000 genes, 80/20 split, and achieves 78.17%; 84.51% is unverified).
- S1–S13 literature findings: Reported by the audit report (`REPORTED-SRC`), not independently inspected by Antigravity during repository initialization.

### 1.4 Proposed But Not Approved
- Proposed Study Framing: Option A (Methodological Rigor and Confounding Study) + Option B (Within-Type Grading) + Option C (Conditional Prognostic Score).
- Candidate Hypotheses: H1 (Confounding Degradation), H2 (Within-Type Discriminability), H3 (Independent Prognostic Utility).
- Dual-label strategy introducing WHO CNS5-approximate molecular grades alongside historical baselines.
- Proposed external validation on CGGA primary tumors (`mRNAseq_693`, `mRNAseq_325`).
- Multivariable Cox proportional hazards modeling (strictly conditional on event counts and data feasibility).

### 1.5 Currently Unknown
- Schema and column names of raw clinical TSVs on Google Drive.
- Expression matrix scale (`star_tpm` raw TPM vs `log2(TPM + 1)`).
- Count of unique biological patients and duplicate aliquots within the 708 rows.
- Count of primary untreated solid tumors (`-01`) vs recurrent (`-02`) or normal (`-11`) tissue.
- Survival event totals, follow-up times, and censoring rates in TCGA clinical files.
- Original script or notebook generating the Phase-I reported 84.51% accuracy.
- Patient overlap between CGGA `mRNAseq_693` and `mRNAseq_325`.
- Dedicated GPU resources, team member commitments, and submission deadlines.

---

## 2. Technical Pipeline & Implementation Inventory

The following technical components reflect the exact state of the implementation as observed in the supplied Colab Python script export (`01_understanding` §3, `02_audit` §1):

| Pipeline Stage | Implementation Details | Evidence Classification | Status & Assessment |
|---|---|---|---|
| **Data Ingestion** | Reads TCGA-LGG and TCGA-GBM clinical files and `star_tpm` expression matrices from Google Drive. Merges into 708 rows. | `VERIFIED-CODE` (referenced in script) | Functional, but relies on hardcoded Google Drive paths. |
| **Label Derivation** | Grade II & III derived by regex/mapping on `primary_diagnosis.diagnoses` strings. All TCGA-GBM cases assigned Grade IV. | `VERIFIED-CODE` | **Flawed (W2).** Derives grade from historical diagnosis strings. `tumor_grade` mostly unrecorded. |
| **Cohort Size** | 708 samples total: Grade II: 320, Grade III: 213, Grade IV: 175. | `REPORTED` | Unaudited for replicate aliquots or non-primary tissue. |
| **Gene Pre-filtering** | Drops all-zero genes, drops genes with mean TPM $< 1$, filters to top 5,000 genes by variance. | `VERIFIED-CODE` | **Flawed (W3).** Executed globally on all 708 rows prior to split (preprocessing data leakage). |
| **Data Splitting** | Stratified 80/20 train/test split (566 train, 142 test). | `VERIFIED-CODE` | **Flawed (W4).** Random stratified split, not grouped by patient barcode (`bcr_patient_barcode`). |
| **Class Balancing** | SMOTE (Synthetic Minority Over-sampling Technique) applied to training set. | `VERIFIED-CODE` | **Flawed (W8, W9).** Applied to high-dimensional continuous gene space; interacts poorly with Keras `validation_split`. |
| **Machine Learning Models** | Random Forest, XGBoost, Support Vector Machine (RBF kernel), Deep Neural Network (DNN), Contrastive Encoder + KNN. | `VERIFIED-CODE` | Implemented. Hyperparameters partially tuned via Optuna on training set. |
| **Feature Selection (SHAP)** | TreeExplainer on Random Forest, top 150 genes selected, used to train "Golden" DNN. | `VERIFIED-CODE` | **Flawed (W7, W13).** SHAP computed on SMOTE-augmented training set; class index `[1]` hardcoded; selection inside CV loop lacks fold independence. |
| **Model Ensembling** | Soft-voting ensemble and hand-weighted ensemble (`0.30 RF + 0.30 XGB + 0.15 SVM + 0.25 DNN`). | `VERIFIED-CODE` | **Flawed (W6).** Weights hand-selected after inspecting held-out test performance. |
| **Tabular Foundation Model** | TabFM code cell written. | `VERIFIED-CODE` | **In Progress / Incomplete.** No execution outputs or results recorded. |

---

## 3. Experimental Results & Performance Claims

### 3.1 Saved Results from Colab Run (REPORTED, Not Independently Reproduced)
The following metrics were recorded in the audit as reported historical notebook results (`01_understanding` §3, `02_audit` §1):

| Model Architecture | Reported Overall Accuracy | Macro-F1 / Balanced Acc | Grade IV Recall | Grade II vs III Accuracy | Evidence Status |
|---|---|---|---|---|---|
| **Support Vector Machine (SVM, RBF)** | 80.99% | UNKNOWN | ~0.97 | UNKNOWN | `REPORTED` |
| **Random Forest (Default)** | 78.17% | UNKNOWN | ~0.97 | 62% – 71% (subset) | `REPORTED` |
| **Optuna-Tuned Random Forest** | 77.46% | UNKNOWN | ~0.97 | UNKNOWN | `REPORTED` |
| **Optuna-Tuned XGBoost** | UNKNOWN | UNKNOWN | ~0.97 | UNKNOWN | `REPORTED` |
| **Deep Neural Network (DNN)** | UNKNOWN | UNKNOWN | ~0.97 | UNKNOWN | `REPORTED` |
| **SHAP Top-150 "Golden" DNN** | UNKNOWN | UNKNOWN | ~0.97 | UNKNOWN | `REPORTED` |
| **Contrastive Encoder + KNN** | UNKNOWN | UNKNOWN | ~0.97 | UNKNOWN | `REPORTED` |
| **Weighted Ensemble** | UNKNOWN | UNKNOWN | ~0.97 | UNKNOWN | `REPORTED` |

### 3.2 Critical Discrepancy: Phase-I Report vs. Codebase (W12)
A major inconsistency exists between the official Phase-I project report and the actual Python code (`01_understanding` §3, `02_audit` §1, §6):

| Dimension | Phase-I Formal Written Report | Actual Python Script Export (`.py`) | Discrepancy Analysis |
|---|---|---|---|
| **Data Normalization** | Log transformation: `log2(TPM + 1)` claimed | No log transformation applied; raw TPM filtered | Report claims transformation absent in code. |
| **Feature Selection** | Hybrid Mutual Information + LASSO + Random Forest; top 500 genes | Simple variance threshold filtering to top 5,000 genes | Entirely different feature selection pipeline. |
| **Data Partitioning** | 80% train / 10% validation / 10% test | 80% train / 20% test (single split) | Partitioning scheme does not match. |
| **Headline RF Accuracy** | **84.51%** | **78.17%** | **84.51% is unverified and not reproduced.** |
| **Bibliography & Schedule** | Text cites 60 studies, but lists only 22 references; timeline dates inconsistent | N/A | Scholarly hygiene issues in written report. |

> [!WARNING]
> The headline figure of **84.51%** cannot be cited as a baseline or reproduced from the provided codebase. It must be marked `UNVERIFIED` and investigated under Milestone M0/P0.

---

## 4. Verified Code Deficiencies & Vulnerabilities

Direct inspection of the `.py` script confirmed seven distinct technical flaws (`01_understanding` §3, `02_audit` §6):

1. **Global Preprocessing Leakage (W3):** Expression variance filtering (60,660 $\rightarrow$ 5,000) and zero/low-expression drops run across the entire 708-sample matrix prior to splitting.
2. **Ungrouped Patient Splitting (W4):** `train_test_split(stratify=y)` splits rows independently. If patients contribute multiple tumor aliquots, data from the same biological subject leaks across train and test sets.
3. **Pervasive Test-Set Reuse (W6):** The identical 142-sample test set is used to compare all models, select Optuna configurations, evaluate SHAP subsets, and tune ensemble weights.
4. **Circular Cohort Confounding Diagnostic (W10):** The diagnostic cell designed to check cohort balance constructs the variable `cohort` directly from `Target_Grade` (e.g., if Grade IV, assign to GBM; else LGG), rendering the confounding diagnostic entirely circular and meaningless.
5. **Hardcoded SHAP Multi-Class Index (W13):** The code computes SHAP values with `shap_values[1]`, arbitrarily extracting feature attributions for a single class rather than across all classes.
6. **Unverified Patient Uniqueness:** The code labels rows as "patients", but barcode trimming (`bcr_patient_barcode`) and duplicate checks are absent.
7. **Code Cleanliness & Deprecations (W18):** Comments refer to an undefined variable `y_final`; XGBoost uses deprecated `use_label_encoder=False`.

---

## 5. Resource & Environmental Inventory

| Resource Category | Item & Specification | Availability Status | Evidence Label |
|---|---|---|---|
| **Primary Data** | TCGA-LGG clinical & `star_tpm` files | Stored on Google Drive (referenced in `.py`) | `VERIFIED-CODE` |
| **Primary Data** | TCGA-GBM clinical & `star_tpm` files | Stored on Google Drive (referenced in `.py`) | `VERIFIED-CODE` |
| **Compute Platform** | Google Colaboratory (GPU type/quota unclear) | Used for Phase-I execution | `REPORTED` |
| **Collaboration Infrastructure** | Multi-Model AI Review Protocol, Context Pack, Prompts & Archive | Established in repository | `VERIFIED-CODE` |
| **External Validation Data** | Chinese Glioma Genome Atlas (CGGA `mRNAseq_693`, `mRNAseq_325`) | Download required; access terms and suitability unchecked | `UNVERIFIED` |
| **Normal Tissue Data** | GTEx / recount-brain normal brain transcriptomes | Proposed by team; integration unchecked | `UNVERIFIED` |
| **Personnel & Team** | Team size, individual member roles, author split | Unknown | `UNKNOWN` |
| **Compute Budget** | Dedicated GPU hardware / HPC cluster access | Unknown | `UNKNOWN` |
| **Project Milestones** | Formal capstone deadlines and Q1 submission dates | Unknown | `UNKNOWN` |
| **Domain Expertise** | Consulting neuropathologist or clinical bioinformatician | Unknown | `UNKNOWN` |

---

## 6. Summary of Progress by Stage

- **Discovery (Prompt 1):** Complete ([incoming/01_research_project_understanding.md](incoming/01_research_project_understanding.md)).
- **Independent Scientific Audit (Prompt 2):** Complete ([incoming/02_independent_scientific_audit.md](incoming/02_independent_scientific_audit.md)).
- **Multi-Model Collaboration Infrastructure:** Complete. [AI_CONTEXT_PACK.md](AI_CONTEXT_PACK.md), [MULTI_MODEL_REVIEW_PROTOCOL.md](MULTI_MODEL_REVIEW_PROTOCOL.md), [MULTI_MODEL_REVIEW_PROMPT.md](MULTI_MODEL_REVIEW_PROMPT.md), and [docs/ai_reviews/](docs/ai_reviews/README.md) established.
- **Phase II Execution Planning (Prompts 3–6):** Not started. Blocked pending Priority P0 data audit and supervisor checkpoint.
- **Active Milestone:** **Milestone M0: Read-Only Data Feasibility Audit (Pending)**.
- **Model Development / Training:** Halted. Prohibited until Milestone M0 is completed and verified.
