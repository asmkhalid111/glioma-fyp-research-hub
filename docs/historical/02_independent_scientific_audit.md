# FYDP Glioma RNA-seq Project — Prompt 2: Independent Scientific Audit & Literature Evaluation

**Document purpose:** Self-contained context file. A new AI assistant or a supervisor can read this and continue the work without the earlier chat.
**Project:** RNA-seq classification of diffuse glioma grades (II, III, IV) using TCGA-LGG and TCGA-GBM, with Phase II aimed at a Q1 journal.
**Stage:** Prompt 2 (independent scientific audit). Prompt 1 (project understanding) is summarized in Section 1. Prompts 3–6 (execution plan, milestones, decision tree, handoff) are NOT done.
**Date of audit:** 13 Nov 2026.

---

## 0. How to use this file and the status legend

**Standing rules for any assistant that reads this file:**
1. Do not invent references, sample counts, column names or file names. If something is unknown, write UNKNOWN.
2. Treat every item marked UNVERIFIED or REPORTED as a lead, not a fact.
3. Do not silently accept the strategy below. Challenge it when the evidence supports doing so.
4. Do not change the research direction without stating what is kept, corrected, removed or reconsidered.
5. Check every citation (DOI, authors, year) before it goes into a manuscript.

**Status labels:**

| Label | Meaning |
|---|---|
| VERIFIED-CODE | Seen directly in the supplied `.py` file |
| VERIFIED-SRC | Supported by a source retrieved during this audit (Section 11) |
| PARTIAL | A source supports part of the claim only |
| REPORTED | Told to the assistant by the user or from documents not visible to it |
| UNVERIFIED | Plausible, but not confirmed |
| ASSUMED | Working assumption |
| PROPOSED | A recommendation, not a fact |

**Not visible to the auditor:** raw data, `.ipynb` outputs, the Phase-I report, the supervisor's actual words, the Prompt 1/3/4/5/6 texts, and the contents of `FYDP_Research_Audit_and_Phase_II_Roadmap.md` except as quoted in the chat.

---

## 1. Project snapshot (from Prompt 1)

- **Data:** TCGA-LGG and TCGA-GBM clinical files plus `star_tpm` expression files on Google Drive. Both look like UCSC Xena / GDC-hub exports (UNVERIFIED).
- **Labels:**
  - LGG grades come from `primary_diagnosis.diagnoses` strings, because `tumor_grade` is mostly "Not Reported" (REPORTED).
  - Mapping: anaplastic astrocytoma and anaplastic oligodendroglioma → Grade III. Astrocytoma NOS, oligodendroglioma NOS and Mixed glioma → Grade II.
  - Every TCGA-GBM sample → Grade IV (VERIFIED-CODE).
- **Master dataset:** 708 rows. Grade II 320, Grade III 213, Grade IV 175 (REPORTED).
- **Pipeline (VERIFIED-CODE):**
  1. Drop all-zero genes, then genes with mean TPM below 1, then keep the top 5,000 by variance. This is done on all data before the split.
  2. Stratified 80/20 split (566 train, 142 test).
  3. SMOTE on the training set.
  4. Models: RF, XGBoost, SVM (RBF), DNN, contrastive encoder + KNN.
  5. Out-of-fold-free SHAP on RF, top 150 genes, then a "golden" DNN.
  6. Optuna-tuned RF and XGB.
  7. Soft-voting and hand-weighted ensembles.
  8. TabFM cell, unfinished.
- **Reported results (not reproduced):**
  - SVM 80.99%, RF 78.17%, tuned RF 77.46%.
  - Grade II vs III only: about 62–71%.
  - Grade IV recall about 0.97.
- **Phase-I report vs code:** the report claims log2(TPM+1), MI+LASSO+RF selection, top 500 genes, an 80/10/10 split and 84.51% RF accuracy. The code shows none of these and gives 78.17%. The 84.51% figure is not reproduced (REPORTED).
- **Phase II candidate objectives (none approved):**
  1. Grade II/III discrimination with external validation on CGGA.
  2. Multiclass II/III/IV with the cohort contribution quantified.
  3. Tumor vs normal detection (teammate suggestion).
  4. Stable annotated genes.
  5. Cox survival analysis (supervisor and teammate suggestion).
- **Open decisions:** label target (historical vs CNS5), the role of normal samples, whether Cox is core or optional, deadline and venue, and the team and compute available.

---

## 2. Executive verdict

1. **The problem is real but not novel.** TCGA glioma grading with ML is a crowded area (Section 5). A paper that only classifies TCGA grades more cleanly would be incremental.
2. **The target label is the biggest scientific problem.**
   - WHO CNS5 (2021) grades gliomas within molecular types. Grade is no longer purely histological (VERIFIED-SRC: S1–S4).
   - A Grade II/III/IV label built from 2007/2016-era histology strings is a legacy target.
   - It can still serve as a documented baseline. It should not be the main clinical claim.
3. **Cohort–grade confounding is critical.**
   - Grade IV comes only from TCGA-GBM. Grades II and III come only from TCGA-LGG (VERIFIED-CODE).
   - Grade IV performance (about 0.97 recall) cannot be separated from "which project/cohort or IDH group is this".
   - The existing cohort diagnostic is circular, because it derives the cohort from the grade (VERIFIED-CODE).
4. **Reported numbers are not trustworthy as they stand.** There is preprocessing leakage, test-set reuse, a likely duplicate-patient problem, and a SMOTE/validation-split artifact (Section 6, W1–W12).
5. **Grade II vs III at about 62–71% is not necessarily a failure.** Published comparators in the same range exist (Section 5), which suggests a real ceiling for this task. A paper built on "we beat 70%" is weak. A paper that explains why the ceiling exists is stronger.
6. **The most defensible Phase II is a rigor and confounding paper**, possibly with a prognostic layer. It should not be a "best accuracy" paper. Details are in Sections 9–10.

---

## 3. Corrections to earlier assistant claims

The earlier Phase II roadmap made claims that this audit re-checked.

| Earlier claim | Audit result |
|---|---|
| "Many papers report ~80–90% for TCGA glioma grading" | **PARTIAL.** A systematic review (S7) of 85 glioma-grade ML studies reports mean best accuracy 0.89 ± 0.09. Those studies are mostly **MRI-based**. RNA-seq-specific figures remain UNVERIFIED. |
| "Ceccarelli annotations cover IDH, 1p/19q for both LGG and GBM" | **VERIFIED-SRC (S10, S11).** The study covers 1,122 TCGA diffuse gliomas (516 LGG, 606 GBM). Its supplement carries molecular data. The exact column names are UNKNOWN. |
| "CGGA contains Grades II, III and IV with IDH, 1p/19q and OS" | **PARTIAL (S5, S6).** A grade field and survival fields exist. But labels follow WHO 2007/2016, not CNS5. Datasets include recurrent and secondary tumors. IDH is called from RNA-seq reads. 1p/19q is predicted from expression. |
| "Many IDH-wt TCGA-LGG Grade III become grade 4 under CNS5" | **UNSUPPORTED as phrased.** CNS5 needs microvascular proliferation/necrosis, or TERT promoter mutation, EGFR amplification or +7/−10 (S1–S3). Some IDH-wt lower-grade astrocytomas without those features are indolent (S4). Ceccarelli reports an IDH-wt "PA-like" LGG subgroup with favorable survival (S10). TCGA tables show about 92 IDH-wt Grade II/III cases (Section 5.3). How many are molecular GBM is unknown. |
| "Survival is the headline result" | **Premature.** Event counts, follow-up quality and confounding by IDH/age are unchecked. |
| "Normal brain samples should be deprioritized" | **Retained as a judgment.** It conflicts with the audit document, so it needs a decision (Section 9, Q5). |

---

## 4. Module A — Research foundation

### A1. Is the problem meaningful?
- Grade II vs III has known histological interobserver variability (S1, S10, S4). A reproducible molecular readout has clinical logic.
- CNS5 now grades within molecular type (S2). A "transcriptomic grade" is meaningful only if it is defined against that framework, or it is explicitly framed as a legacy-label study.
- Tumor-grade prediction from RNA alone has limited clinical need. IDH, 1p/19q, CDKN2A/B, TERT, EGFR and +7/−10 are already tested directly. RNA must offer something extra (see A3).

### A2. Is the problem established?
- Yes. A systematic review (S7) found 85 glioma-grade ML studies up to early 2021, with poor reporting quality and little external validation. The most recent literature is not covered by this audit.

### A3. Are objectives and hypotheses testable?
- No formal hypotheses exist (UNVERIFIED). Five candidate objectives would overload one paper.
- **Testable hypotheses that would fit:**
  - **H1.** After removing cohort and IDH confounding, transcriptome-based grading performance drops by a measurable amount relative to the naive TCGA-LGG/GBM setup.
  - **H2.** Within IDH/1p19q-defined groups, Grade II vs III is separable above chance under patient-grouped nested CV, and the result holds on CGGA.
  - **H3.** RNA-derived risk adds prognostic information beyond age, IDH and assigned grade (conditional on event counts).
- **Retain:** the multi-model comparison as a baseline.
- **Reconsider:** tumor-vs-normal detection, which is nearly trivial and prone to batch shortcuts.

### A4. Can the study design answer the questions?
- **Currently no.** Reasons: grade and cohort are perfectly aligned, there is no external test, and the labels are legacy.
- It can answer them after: relabeling, IDH/1p19q stratification, patient-level grouping, CGGA external testing, and a pre-specified analysis plan.

---

## 5. Module B — Literature and novelty

> **Method limit.** This is a targeted check from one search batch, not a systematic review. A full search (PubMed/Scopus/Embase) with a written protocol is still required before any novelty claim goes into a paper.

### 5.1 Findings

| Topic | Finding | Source | Status |
|---|---|---|---|
| CNS5 grading rules | IDH-mutant astrocytoma is graded 2/3/4. CDKN2A/B homozygous deletion gives grade 4. IDH-wt diffuse astrocytic glioma with TERT promoter mutation, EGFR amplification or +7/−10 is glioblastoma grade 4. "Anaplastic" terms are retired. Grading is within type. Oligodendroglioma is IDH-mutant with 1p/19q codeletion, grade 2 or 3. | S1–S4 | VERIFIED-SRC |
| Histologic grade II/III is subjective | Mitosis and Ki-67 thresholds are not firmly defined. High interobserver variability is noted. | S1, S3, S4, S10 | VERIFIED-SRC |
| Existing ML grade-prediction literature | 85 studies, mean best accuracy 0.89. Only 5 had external validation, and mean accuracy there was lower (0.82). TRIPOD adherence averaged 44.5%. Mostly imaging. | S7 | VERIFIED-SRC |
| TCGA Grade III vs IV with SMOTE and classical ML | A 2024 study reports about 77–83% accuracy using four genes. It is an internal TCGA split with no CGGA validation. | S8 | VERIFIED-SRC |
| LGG II vs III with deep learning on images plus molecular data | 73.95% ± 3.73% for LGG II vs III, 5-fold CV, TCGA only. | S9 | VERIFIED-SRC |
| RNA-based IDH/1p19q prediction | Predicting IDH and 1p/19q from RNA-editing features reached AUC about 0.99. The authors state the models cannot separate Grade II from III astrocytoma, and they had no independent validation cohort. | S12 | VERIFIED-SRC |
| TCGA → CGGA external validation with survival-based subtypes | A TCGA-trained model had CGGA RNA-seq C-index 0.79 (N = 970). | S13 | VERIFIED-SRC |
| TCGA pan-glioma resource | 1,122 gliomas. The IDH-wt "PA-like" LGG subgroup has favorable survival. IDH is the dominant driver of transcriptome clustering. | S10, S11 | VERIFIED-SRC |

### 5.2 Implications for novelty
- **Distinguishing the types of novelty:**
  - *New question:* "How much of TCGA glioma grading accuracy is cohort/IDH artifact, and what survives under CNS5-aligned labels and external testing?" This is potentially new, but a systematic search is needed to confirm.
  - *New method:* none proposed. The ensembles, contrastive learning and TabFM are not novel contributions.
  - *New dataset application:* low novelty.
  - *Improved implementation:* nested grouped CV and leakage control are good practice, and reviewers will expect them. They are necessary but not sufficient.
- **Do not claim novelty merely because** contrastive learning, TabFM or a particular ensemble has not been tried on this dataset (UNVERIFIED either way, and not a scientific contribution by itself).
- **Likely reviewer objections:** a legacy label, TCGA-only data, the cohort-grade link, a "just another ML comparison" feel, and a small Grade II/III advantage over comparators.

### 5.3 Class-size evidence for relabeling (REPORTED from S12; older TCGA subset of 638 samples)

| Group | G2 | G3 | G4 |
|---|---|---|---|
| IDH-wt (n=225) | 20 | 72 | 127 |
| IDH-mut codel (n=163) | 91 | 72 | 0 |
| IDH-mut non-codel (n=250) | 127 | 112 | 8 |

Consequences:
- Within-type Grade II vs III has workable sizes (about 160–240 per IDH-mutant group).
- Under CNS5 the IDH-mutant grade 4 class will be small (it depends on CDKN2A/B status).
- Grade IV is mostly IDH-wt, so "grade IV vs rest" is largely "IDH-wt vs IDH-mutant".
- Counts in your own files may differ and must be recomputed.

---

## 6. Module C — Methodological validity (weakness register)

Severity: **C** critical, **H** high, **M** medium, **L** low.

| ID | Sev | Weakness | Evidence | Effect | Fix |
|---|---|---|---|---|---|
| W1 | C | Grade and cohort perfectly aligned (II/III = LGG, IV = GBM) | VERIFIED-CODE | The model can learn the project or IDH status instead of grade | CNS5/IDH stratification, CGGA external test, an IDH-wt LGG vs GBM test |
| W2 | C | Legacy histologic labels; NOS/Mixed forced into Grade II | VERIFIED-CODE | Label noise and an obsolete target | Dual-label design; drop or analyze NOS and Mixed separately |
| W3 | H | Gene filters (all-zero, mean ≥ 1, top-5000 variance) are computed on the full data before the split | VERIFIED-CODE | Mild but reportable leakage | Move into the CV pipeline (training folds only) |
| W4 | H | Splits are not grouped by patient; multiple samples per patient are possible | UNVERIFIED | Leakage if duplicates exist. Inflated scores | Aggregate by TCGA patient barcode (first 12 characters); use grouped CV |
| W5 | H | Sample types not filtered. Recurrent (`-02`) or normal (`-11`) samples may be present and could be labeled Grade IV or LGG grades | UNVERIFIED. Counts are suggestive (LGG 533 vs 516 patients in S10) | Label noise and duplicates | Keep primary tumors (`-01`) only; check the sample-type code |
| W6 | H | One 20% test set reused for model selection, ensemble weights (0.30/0.30/0.15/0.25) and "which ensemble is best" | VERIFIED-CODE | Optimistic bias; weights tuned on test | Nested CV; freeze the final model before touching external data |
| W7 | H | The top-150 SHAP genes are chosen on the whole training set, then the Optuna CV runs inside the same training set on those genes | VERIFIED-CODE | CV scores are optimistic (selection leakage inside CV). The held-out test remains clean | Put selection inside each fold |
| W8 | H | SMOTE appends synthetic rows after real ones. Keras `validation_split=0.1` takes the last 10% unshuffled, so validation is largely synthetic minority samples | VERIFIED-CODE (reasoning from library behavior; confirm by checking class mix of the validation slice) | Early stopping and "validation" are unreliable. Synthetic samples correlate with their training neighbors | Split first, apply SMOTE inside the training folds only, use a real-sample validation set |
| W9 | H | SMOTE on 5,000 (or 150) continuous genes with n of a few hundred | Method concern | Synthetic points may not be biologically plausible. Gains are unproven | Compare none vs class weights vs SMOTE inside folds |
| W10 | H | Circular cohort diagnostic: `cohort` is derived from `Target_Grade` | VERIFIED-CODE | The diagnostic is meaningless | Use a true cohort/project variable plus CNS5 labels |
| W11 | M | Possible scale confusion. Xena-style `star_tpm` files are often already log2(TPM+1), but the code treats values as raw TPM when filtering (mean ≥ 1) and when computing variance | UNVERIFIED (check the file header and value range) | The filter threshold has a different meaning. The Phase-I report says log2 | Inspect the file and document the transformation |
| W12 | M | The 84.51% Phase-I result is not reproducible from the shared code | REPORTED | Credibility risk | Find the source notebook or delete the claim |
| W13 | M | SHAP class index `[1]` hard-coded; SHAP computed on a model trained on SMOTE data; no stability analysis | VERIFIED-CODE | Importance is unstable. Ensembl IDs are not mapped to gene symbols | Out-of-fold SHAP, selection frequency, gene symbols |
| W14 | M | Too many model families on about 700 samples (RF, XGB, SVM, DNN, contrastive, TabFM, ensembles) | VERIFIED-CODE | Multiple-comparison risk. Complexity without justification | Fewer models. Pre-specify the primary model |
| W15 | M | The contrastive/KNN model trains on full-batch embeddings with a small encoder and a fixed 150 epochs | VERIFIED-CODE | Hard to justify as a contribution | Keep only if it wins under nested CV |
| W16 | M | Class imbalance is handled for training only; reported accuracy is not the primary metric | VERIFIED-CODE | Accuracy hides weak Grade II/III performance | Report balanced accuracy, macro-F1, MCC, per-class recall, AUROC/AUPRC, with bootstrap CIs |
| W17 | M | Seeds fixed but no repeated CV; results from a single split | VERIFIED-CODE | Wide uncertainty is not shown | Repeated nested CV, report variance |
| W18 | L | `y_final` mentioned in a comment but never defined. `use_label_encoder` is deprecated | VERIFIED-CODE | Reproducibility hygiene | Clean the code |

---

## 7. Module D — Data and resources

### D1. TCGA-LGG / TCGA-GBM (Xena/GDC STAR TPM)
- **Suitability:** large, public, with matched molecular data via Ceccarelli (S10, S11).
- **Issues:**
  - The grade for LGG was derived from diagnosis strings, which is error-prone (W2).
  - The scale (log vs raw) is unknown (W11).
  - Sample types and duplicates are unaudited (W4, W5).
  - TCGA tissue source sites and batch structure are unaddressed. Platform and tissue-source effects can masquerade as grade signal.
  - The Ceccarelli RNA-seq subset (667 profiles, S10) is older than the GDC STAR re-processing. Matching IDs requires care.
- **Access/ethics:** open-access expression and clinical data are generally available. Controlled data (such as some mutation or copy-number files) may need dbGaP approval. Exact terms are UNVERIFIED and must be checked.

### D2. CGGA (external)
- **Verified facts (S5, S6):**
  - mRNAseq_693 has 282 primary LGG, 161 recurrent LGG, 140 primary GBM and 109 recurrent GBM.
  - mRNAseq_325 has 144 primary LGG, 38 recurrent LGG, 85 primary GBM, 24 recurrent GBM and 30 secondary GBM.
  - Clinical fields include histology, grade, age, gender, therapy, survival and censoring.
  - Expression comes from STAR+RSEM. Data are on Illumina HiSeq. Tumors were graded under WHO 2007/2016.
  - IDH is inferred from RNA/WES reads at hotspot positions. 1p/19q codeletion is predicted from expression using a published method (Hu et al. 2017).
- **Concerns:**
  - **Recurrent and secondary tumors are included.** Restrict to primary samples, or treat recurrence separately.
  - **Predicted 1p/19q** from expression is not independent ground truth. Using it as a label for an expression-based model carries circularity.
  - CGGA labels are WHO 2007/2016, not CNS5.
  - A Chinese single-country cohort is a useful external test, but it differs in population and pipeline.
  - **Possible overlap between mRNAseq_693 and mRNAseq_325** is UNVERIFIED. Check IDs before treating them as independent.
  - Different TPM/FPKM units, gene annotations and library prep create domain shift.
  - The CGGA site also lists TCGA expression profiles it re-hosted. Do NOT use those as external data.
  - Access terms (registration, data use agreement) are UNVERIFIED.

### D3. Normal brain (GTEx, recount-brain; teammate suggestion)
- Resource details are UNVERIFIED in this audit.
- Normal-vs-tumor is trivially separable. A GTEx vs TCGA comparison is dominated by batch, post-mortem and processing differences.
- Keep it, at most, as a supplementary out-of-distribution sanity check. Make it a core objective only with a pipeline-harmonized source (for example uniformly reprocessed data) and an explicit batch analysis.

### D4. Inclusion/exclusion criteria (PROPOSED; to be pre-registered)
- Include: adult diffuse glioma, primary untreated tumor, one sample per patient, available RNA-seq, and a defined label.
- Exclude: recurrent and normal samples, duplicate aliquots, NOS/Mixed from the primary analysis (analyze as a sensitivity set), pediatric-type tumors, and cases with failed QC.

---

## 8. Module E — Evaluation and interpretation

| Issue | Why it matters | Recommendation |
|---|---|---|
| Overall accuracy as the headline | Dominated by the easy Grade IV class | Use balanced accuracy, macro-F1, MCC, and per-class recall and AUROC |
| Grade IV recall of about 0.97 | Likely reflects cohort/IDH signal | Show IDH-stratified results and the IDH-wt LGG vs GBM comparison |
| Grade II/III accuracy of about 62–71% on a small subset | A wide confidence interval is likely (test n is only a few dozen per class) | Bootstrap 95% CIs from repeated nested CV |
| Single internal split | High variance | Repeated stratified, patient-grouped, nested CV |
| No external validation | The main reviewer objection (S7: only 5 of 85 studies had it) | One locked-model test on CGGA primary tumors |
| SHAP as "biomarkers" | SHAP shows model use, not biology or causation | Stability across folds, gene symbols, enrichment with a proper background, CGGA replication |
| Survival claims | A predicted grade may mainly proxy IDH status and age | Multivariable Cox with age, IDH, 1p/19q and assigned grade. Check proportional hazards. Report the C-index gain with CIs |
| Causal language | Observational, retrospective data | Use "associated with", not "drives" |
| Clinical-use language | Not a validated diagnostic | State it as a research tool and follow TRIPOD+AI |

**Likely overclaim risks:** calling the 150 genes "golden biomarkers", calling the grade IV result "clinical performance", and presenting ensemble gains that come from test-set weighting.

---

## 9. Module F — Missing analyses, corrections, alternatives

### 9.1 Missing experiments and analyses (priority order)
1. **Read-only data audit:** unique patients, sample-type codes, diagnosis × grade × project cross-tab, scale of the expression values, and survival-event counts.
2. **Duplicate handling:** one primary sample per patient.
3. **Molecular annotation join:** Ceccarelli IDH, 1p/19q, subtype, plus copy-number-derived CDKN2A/B deletion, +7/−10 and EGFR amplification where available. TERT promoter coverage is limited in exome data (UNVERIFIED), so the CNS5 relabel will be approximate. Label it "CNS5-approximate".
4. **Confounding experiments:**
   - Predict cohort/project from expression within an IDH-matched subset.
   - Test whether IDH-wt LGG tumors cluster with GBM or with LGG.
   - Train a model on IDH only and see how much of "grade" it explains.
   - Compare Grade IV prediction before and after removing IDH-related variation.
5. **Within-type Grade II vs III** (IDH-mutant astrocytoma, and 1p/19q-codeleted oligodendroglioma) under grouped nested CV.
6. **Baselines:** a simple model such as elastic-net logistic regression, and a "clinical-only" baseline (age, IDH, 1p/19q), so reviewers can see what RNA adds.
7. **External validation on CGGA** with a locked model and a documented harmonization (shared genes, log-transform, rank or ComBat fitted on training data only).
8. **Batch/tissue-source analysis** in TCGA.
9. **Survival/Cox analysis**, conditional on event counts.
10. **Gene-signature stability** and pathway enrichment.
11. **Ablation of imbalance handling** (none vs class weights vs SMOTE in-fold).

### 9.2 Recommended corrections (retain / correct / remove)

| Component | Decision |
|---|---|
| Data loading and merging | **Retain**, add patient and sample-type filters |
| Historical labels | **Retain as a baseline**, document the mapping and its limits |
| CNS5-approximate labels | **Add** as the main analysis if the data support it |
| Gene filtering | **Correct** (inside folds; check scale) |
| 80/20 single split and test-set reuse | **Remove**, replace with nested grouped CV |
| SMOTE | **Correct** (in-fold only) or compare to alternatives |
| RF/XGB/SVM/elastic-net | **Retain** as the main comparison |
| DNN, golden DNN, contrastive KNN | **Retain only if** they win under nested CV. Otherwise supplement |
| Manual ensemble weights | **Remove** |
| Soft-voting ensemble | **Reconsider**, only as a pre-specified, equal-weight comparison |
| TabFM | **Remove from the main paper** (supplementary at most) |
| SHAP | **Correct** (out-of-fold, stable, annotated) |
| Cohort diagnostic | **Rebuild** using a real cohort variable |
| Tumor-vs-normal | **Reconsider**, supplementary unless harmonized data exist |

### 9.3 Alternative research directions (not mutually exclusive)

| Option | Idea | Strength | Risk |
|---|---|---|---|
| **A. Confounding and rigor study** | Quantify how much glioma-grade ML on TCGA reflects cohort/IDH/batch artifact, and what survives CNS5 labels plus CGGA validation | Honest, publishable even if performance drops (the "decision point" scenario) | Needs a clear methodological framing. May suit methods or medical-informatics journals more than clinical ones |
| **B. Within-type transcriptomic grading** | Grade II vs III inside IDH-mutant astrocytoma and oligodendroglioma | Matches CNS5 logic, has workable sample sizes | Likely modest accuracy (the published ceiling is about 70%) |
| **C. Prognostic transcriptomic score** | RNA-derived risk beyond assigned grade, validated on CGGA | Clinically meaningful. A precedent exists for TCGA→CGGA survival models (S13), so novelty must be argued carefully | Depends on event counts. Risk of proxying IDH/age |
| **D. Molecular-grade prediction** | Predict CNS5 molecular-grade features (CDKN2A/B loss, +7/−10) from RNA | A grade-relevant, objective target | Copy-number features from RNA are indirect. Needs literature checking |
| **E. Uncertainty-aware grading** | Calibrated probabilities and abstention for ambiguous Grade II/III cases | Fits label-noise reality | Needs a clear clinical argument |

**Suggested combination (PROPOSED, supervisor to decide):** A as the framing, B as the core classification task, C as a conditional add-on, and D–E as future work.

---

## 10. Unresolved scientific questions

1. What do TCGA clinical files contain beyond `primary_diagnosis` (for example `tumor_grade`, `ajcc`, IDH fields)? Are any grade fields usable?
2. How many TCGA-LGG/GBM RNA-seq samples are recurrent or normal, and how many patients have multiple samples?
3. Are the expression values log-transformed already (W11)?
4. How many IDH-wt LGG cases would be molecular GBM under CNS5, and how many data fields exist to determine this?
5. Which CDKN2A/B, TERT, EGFR and +7/−10 data can be sourced for TCGA, and under what access terms?
6. Do CGGA mRNAseq_693 and mRNAseq_325 overlap? What is the exact primary-tumor grade distribution?
7. How large is the domain shift between TCGA and CGGA, and does it break the classifier?
8. How many survival events exist per CNS5-aligned group?
9. Does a gene signature outperform IDH/1p19q/age alone for grading?
10. What do systematic searches show about prior TCGA→CGGA RNA-based grading work?

---

## 11. Prioritized recommendations

**P0 — before anything else**
1. Run the read-only data audit (Section 9.1 item 1–2).
2. Get the supervisor's decision on: historical vs CNS5 labels, scope (single vs several objectives), and whether Q1 is a hard requirement.
3. Resolve the 84.51% question.

**P1 — core scientific fixes**
4. Relabel with molecular annotations (documented as approximate). Keep the historical baseline.
5. Build the nested, patient-grouped CV pipeline with all learning inside folds.
6. Add the confounding experiments (9.1 item 4).
7. Acquire and harmonize CGGA (primary tumors only), then test once with a locked model.

**P2 — strengthen the paper**
8. Add simple baselines and a clinical-only baseline.
9. Run survival analysis if event counts allow.
10. Run the SHAP stability and pathway analysis.

**P3 — cut or defer**
11. Hand-weighted ensembles, TabFM, normal-brain core objective, extra deep models.

**Pre-registration (PROPOSED):** write the primary endpoint, metrics, models and the CGGA test plan into a short analysis plan before touching CGGA.

**Reporting:** follow TRIPOD+AI. Release code and seeds.

---

## 12. Reliability notes for this audit

- Literature coverage is partial (one search batch). The 2024–2026 literature is not systematically reviewed.
- W8 (SMOTE row order vs `validation_split`) comes from knowledge of library behavior. Confirm with a quick check of the validation slice.
- W11 (log-scale) is a hypothesis from the file naming convention. Check the real values.
- Class-size figures in Section 5.3 come from a 2019 TCGA subset, not from the project's files.
- CGGA access terms, TCGA controlled-data terms and GTEx/recount-brain details were not verified.
- Journal quartile statements from earlier chat turns are UNVERIFIED. Check SCImago before choosing a venue.

---

## 13. Source list (verify each before citing)

| ID | Source | Used for |
|---|---|---|
| S1 | "Adult type diffuse gliomas in the new 2021 WHO Classification", PMC9763975 (2022) | CNS5 grading rules, interobserver variability |
| S2 | "The 2021 WHO Classification for Gliomas and Implications on Imaging Diagnosis: Part 1", J Magn Reson Imaging, doi:10.1002/jmri.28743 | Grading within type, combined molecular grading |
| S3 | "Opinion & Special Article: Glioma Classification", Neurology, doi:10.1212/WNL.0000000000201262 (2022) | CNS5 algorithm, molecular markers |
| S4 | "Grading of adult diffuse gliomas according to the 2021 WHO Classification…", PubMed 34504304 | Grading debate, indolent IDH-wt astrocytoma, molecular markers |
| S5 | CGGA "About" page, cgga.org.cn/about.jsp | Dataset description, labeling methods |
| S6 | CGGA home and download pages (cgga.org.cn; 2019 updates) | Sample counts, expression pipeline |
| S7 | "Machine Learning Models for Classifying High- and Low-Grade Gliomas: A Systematic Review and Quality of Reporting Analysis", Front Oncol, doi:10.3389/fonc.2022.856231 | Prior ML performance, validation and reporting quality |
| S8 | "Integrative analysis of single-cell transcriptomic and multilayer signaling networks in glioma…", Front Genet, doi:10.3389/fgene.2024.1446903 | TCGA Grade III vs IV ML with SMOTE |
| S9 | "Deep Neural Network Analysis of Pathology Images With Integrated Molecular Data for Enhanced Glioma Classification and Grading", Front Oncol, doi:10.3389/fonc.2021.668694 | LGG II vs III accuracy comparator |
| S10 | Ceccarelli et al., Cell 164:550–563 (2016), doi:10.1016/j.cell.2015.12.028 | TCGA pan-glioma cohort, IDH-driven clustering, PA-like LGG, interobserver variability |
| S11 | NCI GDC publication page for Ceccarelli 2016 (gdc.cancer.gov/about-data/publications/lgggbm_2016) | Annotation resource |
| S12 | "RNA editing-based classification of diffuse gliomas: predicting IDH mutation and chromosome 1p/19q codeletion", PMC6929429 (2019) | TCGA IDH/grade cross-tab, limits on II vs III |
| S13 | "Deep learning algorithm reveals two prognostic subtypes in patients with gliomas", PMC9552440 (2022) | TCGA→CGGA external survival-model precedent |

---

## 14. State for the next prompt

- **Completed:** Prompt 1 (understanding), Prompt 2 (this audit).
- **Pending decisions that block later prompts:** label target, objective scope, deadline/venue, team and compute, and the data-audit results.
- **Suggested next step:** run the read-only data audit, bring back the outputs and the supervisor's decisions, then proceed to the execution plan (Prompt 3) and milestones.
- **Instruction for any assistant reading this file:** continue from Section 11. Do not re-propose the methods marked "remove" without new evidence. Mark any new claim with a status label from Section 0.
