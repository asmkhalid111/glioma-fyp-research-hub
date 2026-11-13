# EVIDENCE_REGISTER.md — Formal Empirical & Literature Evidence Register

> **Document Status:** Active Reference Registry  
> **Source Documents:** [incoming/01_research_project_understanding.md](incoming/01_research_project_understanding.md), [incoming/02_independent_scientific_audit.md](incoming/02_independent_scientific_audit.md)  
> **Epistemic Taxonomy:** All entries carry strict status labels (`VERIFIED-CODE`, `REPORTED-SRC`, `PARTIAL`, `REPORTED`, `UNVERIFIED`, `ASSUMED`, `PROPOSED`, `UNKNOWN`).

---

## 1. Verified Code-Level Evidence (`VERIFIED-CODE`)

The following findings were directly observed and confirmed in the exported Colab Python script (`01_understanding` §3, `02_audit` §1, §6):

| Observation ID | Pipeline Location | Verified Code Behavior | Methodological Implication |
|---|---|---|---|
| **CODE-01** | Label Mapping | Grade II and III are derived exclusively by string matching on `primary_diagnosis.diagnoses`. | Unreliable proxy for histological grade; ignores molecular classification. |
| **CODE-02** | Label Mapping | Every TCGA-GBM sample is unconditionally assigned Grade IV: `y[gbm_mask] = 'Grade IV'`. | Creates 100% cohort-grade alignment in dataset; Grade IV prediction is confounded by cohort and molecular differences (W1). |
| **CODE-03** | Preprocessing | Low-expression filtering (mean TPM $< 1$, zero-drops) and top-5,000 variance selection run globally on all 708 samples before data splitting. | Information leakage from test fold into training fold (W3). |
| **CODE-04** | Partitioning | Data split using `train_test_split(X, y, test_size=0.20, stratify=y)`. No patient grouping. | Possible data leakage across splits if patients have multiple aliquots (W4). |
| **CODE-05** | Balancing | `SMOTE()` applied to training set prior to Keras neural network training. | High-dimensional synthetic interpolation; invalid validation slices (W8). |
| **CODE-06** | Test Reuse | Test set (`X_test`, `y_test`) evaluated across RF, XGB, SVM, DNN, tuned variants, and ensemble weight searches. | Severe multiple-testing bias; test set no longer provides an unbiased generalization estimate (W6). |
| **CODE-07** | Ensemble Weights | Hand-crafted weights `[0.30, 0.30, 0.15, 0.25]` applied to ensemble components. | Post-hoc manual optimization on the evaluation set (W6). |
| **CODE-08** | Diagnostic | Cohort check builds variable `cohort` by inspecting `Target_Grade`: `df['cohort'] = df['Target_Grade'].apply(...)`. | Cohort diagnostic is circular and scientifically invalid (W10). |
| **CODE-09** | Feature Selection | SHAP `TreeExplainer` computed on SMOTE training set with hardcoded index `shap_values[1]`. | Class attribution arbitrarily restricted to one class; lacks stability analysis (W13). |
| **CODE-10** | Hygiene | Comment refers to undefined `y_final`; XGBoost uses deprecated `use_label_encoder=False`. | Code cleanliness issue; non-critical (W18). |

---

## 2. Methodological Weakness Register (W1 – W18)

The 18 methodological weaknesses identified during the scientific audit (`02_audit` §6):

| ID | Sev | Weakness Summary | Direct Evidence | Methodological Effect | Required Remediation |
|---|---|---|---|---|---|
| **W1** | **C** | Cohort and grade are perfectly aligned (II/III = LGG, IV = GBM). | `VERIFIED-CODE` | Grade IV prediction confounded by project/IDH identity vs grade biology. | Stratify by IDH/CNS5; test on external CGGA; test IDH-wt LGG vs GBM. |
| **W2** | **C** | Legacy histological labels; NOS and Mixed glioma forced into Grade II. | `VERIFIED-CODE` | Label noise; targets an obsolete grading system. | Adopt dual-label strategy; analyze NOS/Mixed in sensitivity set. |
| **W3** | **H** | Gene filters (zero-drop, mean $\ge 1$, top-5,000 variance) computed before split. | `VERIFIED-CODE` | Preprocessing data leakage; optimistic bias. | Encapsulate all filters strictly within training folds. |
| **W4** | **H** | Data splits are not grouped by patient barcode (`bcr_patient_barcode`). | `UNVERIFIED` | Potential data leakage across folds if repeat aliquots exist. | Group splits by 12-character patient barcode. |
| **W5** | **H** | Sample types unvetted; recurrent (`-02`) or normal (`-11`) tissue may be included. | `UNVERIFIED` | Phenotypic contamination and duplicate patient profiles. | Retain strictly primary solid tumors (`sample_type == '01'`). |
| **W6** | **H** | Single 20% test set reused for model selection, SHAP subsets, and ensemble weights. | `VERIFIED-CODE` | Optimistic bias; overfitting to test partition. | Implement nested, patient-grouped cross-validation ($5 \times 5$). |
| **W7** | **H** | Top 150 SHAP genes selected on whole train set before Optuna tuning. | `VERIFIED-CODE` | Feature selection leakage inside cross-validation loops. | Conduct feature selection strictly within internal CV folds. |
| **W8** | **H** | SMOTE prepends/appends rows; Keras `validation_split=0.1` takes last 10% unshuffled. | `VERIFIED-CODE` | Validation slice composed primarily of synthetic minority rows. | Apply SMOTE in-fold only; validate on genuine biological samples. |
| **W9** | **H** | SMOTE executed on high-dimensional continuous gene space ($P = 5,000$). | Methodological | Biologically implausible synthetic gene profiles. | Compare SMOTE against cost-sensitive weighting and unweighted baseline. |
| **W10** | **H** | Cohort diagnostic derives `cohort` directly from `Target_Grade`. | `VERIFIED-CODE` | Circular diagnostic masks true confounding. | Rebuild diagnostic using authentic TCGA project identifiers. |
| **W11** | **M** | Scale ambiguity: Xena STAR-TPM may already be `log2(TPM + 1)`. | `UNVERIFIED` | Thresholding (mean $\ge 1$) altered if scale is logarithmic. | Inspect raw value distributions and file headers under M0. |
| **W12** | **M** | Phase-I reported RF accuracy of 84.51% is not reproducible from code. | `REPORTED` | Unsubstantiated baseline claim. | Audit historical notebooks or formally retract the 84.51% figure. |
| **W13** | **M** | SHAP class index `[1]` hardcoded; computed on SMOTE data; no stability test. | `VERIFIED-CODE` | Unstable feature ranking; arbitrary class focus. | Pool SHAP across outer folds; report selection frequencies and symbols. |
| **W14** | **M** | Excessive model families evaluated on $N \approx 700$ (RF, XGB, SVM, DNN, KNN, TabFM). | `VERIFIED-CODE` | Multiple testing burden; unjustified architecture complexity. | Focus on pre-specified core suite (RF, XGB, SVM, Elastic-Net). |
| **W15** | **M** | Contrastive KNN trains full-batch embeddings on tiny encoder for 150 epochs. | `VERIFIED-CODE` | Poorly justified novelty claim. | Retain only if statistically superior under nested CV. |
| **W16** | **M** | Class imbalance addressed during training; overall accuracy reported as headline. | `VERIFIED-CODE` | Headline accuracy masks poor Grade II vs III discrimination. | Mandate Balanced Accuracy, Macro-F1, MCC, and AUROC as primary metrics. |
| **W17** | **M** | Single seed and single split; no repeated resampling or confidence intervals. | `VERIFIED-CODE` | Point estimates fail to reflect sampling variance. | Perform repeated nested CV and report 95% bootstrap CIs. |
| **W18** | **L** | Commented undefined variable `y_final`; deprecated XGBoost parameters. | `VERIFIED-CODE` | Minor software hygiene deficiency. | Refactor and clean script. |

---

## 3. Literature Evidence Register (Sources S1 – S13)

> **Notice on Literature Status:** The following 13 sources were identified and reported in the independent scientific audit report (`02_audit` §5.1, §13). They are recorded here as `REPORTED-SRC`. Antigravity has not independently retrieved or inspected their primary texts during repository initialization. They represent the audit's reported citations.

| Source ID | Full Citation / Identifier | Evidence Status | Key Scientific Findings | Relevance & Scope Limits |
|---|---|---|---|---|
| **S1** | "Adult type diffuse gliomas in the new 2021 WHO Classification", *PMC9763975* (2022). | `REPORTED-SRC` | Outlines WHO CNS5 grading taxonomy; highlights mitotic count and cellularity discordance in Grade II/III. | Justifies within-type grading; documents histological discordance. |
| **S2** | "The 2021 WHO Classification for Gliomas and Implications on Imaging Diagnosis: Part 1", *J Magn Reson Imaging*, doi:10.1002/jmri.28743. | `REPORTED-SRC` | Confirms that grading is now performed strictly within molecular types. | Proves that unstratified Grade II/III/IV classification is obsolete. |
| **S3** | "Opinion & Special Article: Glioma Classification", *Neurology*, doi:10.1212/WNL.0000000000201262 (2022). | `REPORTED-SRC` | Formalizes algorithmic workflow for IDH, 1p/19q, and molecular Grade 4 criteria (CDKN2A/B). | Defines the molecular ground truth rules for CNS5-approximate relabeling. |
| **S4** | "Grading of adult diffuse gliomas according to the 2021 WHO Classification…", *PubMed 34504304*. | `REPORTED-SRC` | Documents ongoing debate on indolent IDH-wildtype lower-grade astrocytomas lacking molecular GBM features. | Explains why IDH-wt Grade II/III cases require nuanced, careful treatment. |
| **S5** | Chinese Glioma Genome Atlas "About" Page, *cgga.org.cn/about.jsp*. | `REPORTED-SRC` | Details cohort profiles: WHO 2007/2016 labels, RNA/WES-based IDH calls, Hu et al. predicted 1p/19q. | Establishes clinical properties and known limitations of the proposed CGGA validation cohort. |
| **S6** | Chinese Glioma Genome Atlas Download Portal, *cgga.org.cn*. | `REPORTED-SRC` | Confirms mRNAseq_693 (282 prim LGG, 140 prim GBM) and mRNAseq_325 (144 prim LGG, 85 prim GBM). | Provides sample sizes for proposed external validation. |
| **S7** | "Machine Learning Models for Classifying High- and Low-Grade Gliomas: A Systematic Review…", *Front Oncol*, doi:10.3389/fonc.2022.856231. | `REPORTED-SRC` | 85 ML studies: mean accuracy $0.89 \pm 0.09$; only 5 externally validated (mean drops to 0.82); mean TRIPOD adherence 44.5%. | Evaluated predominantly **MRI radiomics**, not RNA-seq. Demonstrates general reporting/validation gaps. |
| **S8** | "Integrative analysis of single-cell transcriptomic and multilayer signaling networks in glioma…", *Front Genet*, doi:10.3389/fgene.2024.1446903 (2024). | `REPORTED-SRC` | Reports 77%–83% accuracy for TCGA Grade III vs IV using 4 genes and SMOTE; internal split only. | Provides contemporary comparator for TCGA grade discrimination. |
| **S9** | "Deep Neural Network Analysis of Pathology Images With Integrated Molecular Data…", *Front Oncol*, doi:10.3389/fonc.2021.668694. | `REPORTED-SRC` | Reports $73.95\% \pm 3.73\%$ accuracy for LGG Grade II vs III using 5-fold CV on TCGA. | Evaluated **pathology images + molecular data**, NOT bulk RNA-seq. Must not be generalized as RNA ceiling. |
| **S10** | Ceccarelli et al., "Molecular Profiling Reveals Biologically Discrete Subsets and Pathways of Progression in Diffuse Glioma", *Cell* 164:550–563 (2016), doi:10.1016/j.cell.2015.12.028. | `REPORTED-SRC` | 1,122 TCGA gliomas; demonstrates that IDH status is the primary driver of transcriptome clustering; identifies PA-like LGG. | Primary evidence confirming that cohort-IDH differences confound naive TCGA-LGG/GBM classification. |
| **S11** | NCI Genomic Data Commons Ceccarelli Publication Page, *gdc.cancer.gov*. | `REPORTED-SRC` | Public repository for matched molecular annotations (IDH, 1p/19q, subtypes) across TCGA gliomas. | Identified as the annotation source for proposed M0/M2 molecular data joining. |
| **S12** | "RNA editing-based classification of diffuse gliomas: predicting IDH mutation and chromosome 1p/19q codeletion", *PMC6929429* (2019). | `REPORTED-SRC` | Reports IDH/1p19q prediction AUC $\approx 0.99$, but confirms models cannot separate Grade II from III; provides cross-tab. | Provides reference class distributions across IDH and grade. |
| **S13** | "Deep learning algorithm reveals two prognostic subtypes in patients with gliomas", *PMC9552440* (2022). | `REPORTED-SRC` | TCGA-trained transcriptomic model achieved CGGA external validation C-index of 0.79 ($N = 970$). | Precedent demonstrating technical feasibility of TCGA $\rightarrow$ CGGA survival-risk generalization. |

---

## 4. Class-Size Evidence (S12 Reference Distribution)

Reference sample distribution from a 638-sample TCGA glioma cohort (`02_audit` §5.3, S12):

| Molecular Subgroup | Historical Grade II | Historical Grade III | Historical Grade IV | Total Patients |
|---|---|---|---|---|
| **IDH-wildtype** | 20 | 72 | 127 | 219 |
| **IDH-mutant, 1p/19q-codeleted** | 91 | 72 | 0 | 163 |
| **IDH-mutant, non-codeleted** | 127 | 112 | 8 | 247 |
| **Total** | 238 | 256 | 135 | 629 |

> [!NOTE]
> This distribution demonstrates that within IDH-mutant subtypes, Grade II vs III sample sizes are reasonably balanced ($N \approx 160–250$ per type). It also shows that in this historical subset, Grade IV was 94.1% IDH-wildtype ($127 / 135$), confirming that Grade IV prediction is heavily aligned with IDH status and requires controlled investigation.

---

## 5. Corrected Leads & Retracted Claims

Audit of preliminary assertions made prior to formal literature verification (`01_understanding` §0, `02_audit` §3):

| Preliminary Assertion | Audit Assessment | Evidence Status | Corrected Scientific Reality |
|---|---|---|---|
| *"Many papers report 80%–90% for TCGA RNA grading"* | **PARTIAL** | `PARTIAL` (S7) | Systematic review shows $0.89$ mean accuracy for ML grading, but predominantly in **MRI radiomics**. RNA-seq specific figures remain unverified. |
| *"Ceccarelli annotations cover both cohorts"* | **VERIFIED** | `REPORTED-SRC` (S10, S11) | Confirmed in audit: covers 1,122 TCGA diffuse gliomas (516 LGG, 606 GBM). Matching IDs to GDC STAR-TPM requires harmonization. |
| *"CGGA contains Grades II, III, IV with IDH, 1p/19q, OS"* | **PARTIAL** | `PARTIAL` (S5, S6) | Grades follow WHO 2007/2016, not CNS5. Datasets include recurrent/secondary tumors. 1p/19q is predicted from expression (Hu et al.), not direct ground truth. |
| *"Many IDH-wt LGG Grade III become Grade 4 under CNS5"* | **UNSUPPORTED AS PHRASED** | `UNVERIFIED` (S1–S4, S10) | Requires specific molecular criteria (*TERT* promoter, *EGFR* amp, $+7/−10$). Some IDH-wt lower-grade gliomas are indolent or PA-like. Actual count in our cohort is UNKNOWN. |
| *"Survival is the headline result"* | **PREMATURE** | `PROPOSED` | Survival event counts, follow-up duration, and confounding by age/IDH are unverified. Demoted to conditional objective. |
| *"Normal brain samples should be deprioritized"* | **RETAINED AS JUDGMENT** | `PROPOSED` | Confirmed methodological stance: GTEx vs TCGA batch effects dominate biological signals. Kept as supplementary check. |

---

## 6. Unknown & Unverified Information Register

The following items are completely unverified and must not be assumed (`01_understanding` §4, §8, `02_audit` §10):

1. **Exact GDC/Xena Clinical Columns:** The complete schema of clinical TSVs in Google Drive is `UNKNOWN`.
2. **Expression Matrix Value Range:** Whether `star_tpm` values are raw TPM or log-transformed is `UNVERIFIED` (W11).
3. **Biological Duplicates:** The number of unique patients within the 708 rows is `UNVERIFIED` (W4).
4. **Primary vs. Recurrent Counts:** The breakdown of `-01` vs `-02` sample codes is `UNVERIFIED` (W5).
5. **Survival Event Totals:** Total death events and median follow-up times for TCGA-LGG/GBM are `UNVERIFIED`.
6. **Phase-I 84.51% Code Source:** The exact computational script producing 84.51% accuracy is `UNKNOWN`.
7. **CGGA Overlap:** Whether overlap exists between `mRNAseq_693` and `mRNAseq_325` is `UNVERIFIED`.
8. **Controlled Data Restrictions:** Whether matched somatic mutation or copy number files require dbGaP access is `UNVERIFIED`.
9. **Project Hardware & Timeline:** Dedicated GPU resources, team member commitments, and submission deadlines are `UNKNOWN`.
