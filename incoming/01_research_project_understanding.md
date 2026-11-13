# Research Project Understanding Document
**Project:** RNA-seq classification of glioma grades (II, III, IV)
**Stage:** Prompt 1, discovery. No methodology or flowchart is finalized here.

**Sources used:** the Colab `.py` export, the audit and roadmap document (`FYDP_Research_Audit_and_Phase_II_Roadmap.md`), and my earlier Phase II roadmap. The raw data, the `.ipynb` outputs, the Phase-I report and the supervisor's actual words were not visible to me directly. Everything from those is **REPORTED**.

**One correction first.** My earlier roadmap made claims I had not verified. They include "many papers report 80–90%", "Ceccarelli annotations cover both cohorts", "CGGA contains Grades II, III and IV with IDH and 1p/19q", and "many IDH-wt LGG Grade III cases become grade 4." These are all **UNVERIFIED** leads, not findings. My roadmap also deprioritized normal brain samples, while your audit document keeps them. That is an **unresolved conflict**, and it is listed under Section 5.

---

## 1. Project overview

| Item | Content | Status |
|---|---|---|
| Domain | Computational oncology: transcriptomics and ML for diffuse glioma | REPORTED |
| Original problem | Classify glioma into Grade II, III and IV from RNA-seq (TCGA-LGG and TCGA-GBM, STAR-TPM) | VERIFIED in code |
| Grade labels (II/III) | Derived from `primary_diagnosis` strings. The `tumor_grade` field is mostly "Not Reported" | REPORTED |
| Grade IV label | Every TCGA-GBM sample is assigned Grade IV | VERIFIED in code |
| Motivation | Grade II vs III is clinically hard, and an RNA-based aid could help | ASSUMED |
| Gap | Not formally stated. My candidate was "externally validated, interpretable, prognostic." The audit document's candidate was "leakage-controlled Grade II/III with CGGA validation." | PROPOSED |
| Hypotheses | None formally stated | UNVERIFIED |
| Intended contribution | Undecided. Candidates: a rigorous grading framework, a gene signature, tumor detection, survival analysis | Open |

---

## 2. Research objectives

**Original, Phase I:** a three-class grade classifier with SHAP-selected genes and an exploration of several model families.

**Candidate Phase II objectives.** None has been approved.
1. Grade II/III discrimination, leakage-controlled, with external validation (CGGA).
2. Multiclass II/III/IV, with the cohort contribution quantified.
3. Non-neoplastic vs glioma detection (teammate proposal).
4. Stable, annotated candidate genes.
5. Survival analysis with Cox PH (supervisor and teammate suggestion), conditional on data.

**Open tension:** the plan has five objectives, and they could overwhelm a single paper. The audit document already separates core from conditional objectives, and I agree with that direction. The decision belongs to your supervisor.

---

## 3. Current state

### Done (VERIFIED in code, results REPORTED)
- Data loading, label mapping and merging. The result is 708 rows: Grade II 320, Grade III 213, Grade IV 175.
- Filtering from 60,660 genes to 5,000.
- Stratified 80/20 split (566 train, 142 test), then SMOTE.
- Models: RF, XGBoost, SVM, DNN, SHAP top-150 DNN, contrastive encoder + KNN, Optuna-tuned RF and XGB, and several ensembles.
- Saved results (REPORTED, not reproduced): SVM 80.99%, RF 78.17%, tuned RF 77.46%. The Grade II/III-only subset reached 62–71%. Grade IV recall was about 0.97 in all models.

### In progress
- TabFM cell: written, with no results seen. **IN PROGRESS**.
- Phase II planning, including this document.

### Not started
CGGA acquisition, label audit, patient-ID audit, nested grouped CV, gene annotation, pathway analysis, survival audit, and normal-sample data.

### Phase-I report vs code: REQUIRES REVISION
The audit document lists these discrepancies:
- The report says log2(TPM+1), MI+LASSO+RF selection, top 500 genes, an 80/10/10 split and 84.51% RF accuracy. The code shows no log transform, variance filtering to 5,000 genes, an 80/20 split and 78.17% RF accuracy.
- The 84.51% result is **not reproduced**.
- The report's timeline dates are inconsistent, and it cites 60 studies but lists 22 references.

### Code issues I can verify directly in the `.py` file
- The gene filters (all-zero, mean TPM ≥ 1, top-5000 variance) run on all data before the split. This is preprocessing leakage.
- The split is not grouped by patient.
- One test set is reused for many comparisons, including ensemble weights. The `0.30/0.30/0.15/0.25` weights look hand-picked.
- The cohort diagnostic builds `cohort` from `Target_Grade`, so the result is circular.
- The SHAP class index `[1]` is hard-coded.
- The notebook calls rows "patients", but uniqueness is **UNVERIFIED**.
- `y_final` appears in a code comment but is never defined. This is minor.

---

## 4. Available resources

| Resource | Status |
|---|---|
| TCGA-LGG and TCGA-GBM clinical and TPM files on Google Drive | VERIFIED (referenced in code) |
| Colab environment (GPU unclear) | REPORTED |
| CGGA, GTEx, recount-brain | Candidate sources only, UNVERIFIED access |
| Team size and roles | Unknown |
| Deadline, compute and budget | Unknown |
| Domain expert (pathologist or bioinformatician) | Unknown |

---

## 5. Existing methodology and decision conflicts

**Current pipeline:** variance filter → split → SMOTE → RF, XGB, SVM, DNN, contrastive KNN → SHAP top-150 → ensembles.

**Conflicts between the two Phase II proposals now on the table:**

| Topic | Audit document | My earlier roadmap | Needs |
|---|---|---|---|
| Label target | Keep historical labels. Try CNS5 only if the metadata supports it | Make CNS5 the main analysis | Supervisor decision, plus checking what metadata exists |
| Normal samples | Core if the system accepts unknown inputs | Deprioritize | Decide whether tumor detection is an objective |
| Cox PH | Conditional on event count | Central, headline result | Survival data audit |
| Framing | Rigorous framework | Reframe as a prognostic grading signature | Supervisor decision |

I think my earlier "headline" framing for survival was premature. We do not yet know the event counts or follow-up quality.

---

## 6. Major assumptions (ASSUMED, to be examined)

1. NOS and Mixed glioma map to Grade II.
2. The 708 rows are independent patients.
3. TPM values are comparable across TCGA-LGG and TCGA-GBM.
4. CGGA offers a usable external cohort. Its grade coverage, and whether its processing is compatible with TCGA, are unverified.
5. The grade label is trustworthy enough as ground truth. Grade II/III is known to have interobserver variability, which still needs citing.
6. Survival data is adequate for Cox modelling.
7. Q1 acceptance depends on the label definition and external validation. This is my judgment, not a verified editorial criterion.

---

## 7. Potential risks

| Risk | Severity |
|---|---|
| Grade and cohort are perfectly aligned (II/III from LGG, IV from GBM), so grade biology and cohort effects cannot be separated | Critical |
| Obsolete or ambiguous labels (historical grade, NOS mappings) | Critical |
| Cannot reproduce the Phase-I headline result | Critical |
| Preprocessing leakage and test-set reuse make current numbers optimistic | High |
| CGGA performance may collapse (domain shift) | High |
| Scope creep from five objectives | High |
| Small classes after relabeling or deduplication | Medium to high |
| Survival events too few | Medium |
| Normal-vs-tumor shortcut through GTEx-vs-TCGA batch identity | Medium |
| Novelty. Prior work may already match this approach | Medium, UNVERIFIED |

---

## 8. Information requiring verification

I would verify these with primary sources, search and dataset inspection in later steps:
1. Which molecular annotations exist for TCGA-LGG and TCGA-GBM. Candidates are the GDC fields and the Ceccarelli 2016 supplement.
2. What the CGGA cohorts contain: grade distribution, IDH and 1p/19q status, survival, normalization, and whether Grade IV is present.
3. The current WHO CNS5 grading rules, to verify before they are used.
4. Whether TCGA-LGG contains IDH-wildtype cases that CNS5 would regraded as glioblastoma. This matters because it would affect the confounding.
5. The published literature on TCGA glioma grading with RNA-seq. This is needed to judge novelty and to cite the "80–90%" claim, which I currently cannot support.
6. recount-brain and GTEx details (the audit document gives published resource counts as REPORTED only).
7. Participant-level duplicates in the 708 rows.
8. The source of the 84.51% result.

---

## 9. Questions that materially affect the next decision

1. **Target definition:** has your supervisor approved historical grade, CNS5-aligned labels, or both? What exactly did they say about Phase I?
2. **Objectives:** is tumor-vs-normal detection a formal objective? Is Cox PH required, or optional?
3. **Deadline and venue:** is a Q1 journal a hard requirement or a stretch goal, and by what date?
4. **Team and compute:** how many people, what split of work, and is a GPU available?
5. **Data in hand:** do you have the raw clinical TSV files with all columns, and can you access CGGA?
6. **Report vs code:** do you know where the 84.51% experiment lives, or whether it was a different notebook?

I can defer the following: model choice, journal selection and pathway-analysis tools.

---

## 10. Recommended next stage

**Do not start coding or building flowcharts yet.** The next step is a **data and label feasibility audit** (milestones M0 and M1 in your audit document), in this order:

1. **Answer questions 1–2 and 6 above.** They decide the framing.
2. **Run a read-only audit on your existing files.** It should cover unique participants, sample-type codes, a cross-tab of diagnosis, grade and project, and the survival fields with event counts. I can write that code, and it changes nothing in your models.
3. **Run a verification search for items 1–4 in Section 8.** This covers CNS5 rules, the molecular fields available in TCGA, and what CGGA contains. I can do this one topic at a time. I have only one web search per response, so I'd suggest starting with the CGGA and TCGA-annotation question.
4. **Then hold a decision checkpoint with your supervisor** on target definition and scope, before any final methodology or flowchart (Prompt 2).

Do you want to start with the answers to the questions in Section 9, or should I draft the read-only audit script first?
