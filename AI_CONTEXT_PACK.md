# AI_CONTEXT_PACK.md — Portable Research Context Pack

> **Context Provenance & Metadata**  
> - **Pack Version:** 1.0.0  
> - **Generation Date:** 13 November 2026  
> - **Base Git Commit SHA:** `812b94b9a2690989796cf0c9192e0094a9260e54`  
> - **Canonical Source Documents:** [README.md](README.md), [CURRENT_STATE.md](CURRENT_STATE.md), [RESEARCH_BLUEPRINT.md](RESEARCH_BLUEPRINT.md), [DECISION_LOG.md](DECISION_LOG.md), [EVIDENCE_REGISTER.md](EVIDENCE_REGISTER.md), [MILESTONE_REGISTER.md](MILESTONE_REGISTER.md), [DATA_GOVERNANCE.md](DATA_GOVERNANCE.md), [AGENTS.md](AGENTS.md)  
> - **Repository Verification:** Verified against the latest `main` branch of `glioma-fyp-research-hub`. This document is a portable synthesis and does not supersede canonical repository documents.

---

## 1. Project Identity & Research Overview

| Attribute | Specification | Evidence Label |
|---|---|---|
| **Research Title** | Transcriptomic Classification and Confounding Analysis of Adult Diffuse Glioma Grades | `REPORTED` |
| **Domain** | Computational Oncology, Bulk RNA-seq Transcriptomics, Clinical Machine Learning | `REPORTED` |
| **Primary Datasets** | TCGA-LGG (Brain Lower Grade Glioma) & TCGA-GBM (Glioblastoma Multiforme), STAR-aligned TPM | `VERIFIED-CODE` |
| **Target Publication** | Q1 Medical Informatics / Computational Oncology Journal (Proposed Target) | `UNVERIFIED` |
| **Current Sample Size** | 708 merged profiles: Grade II (320), Grade III (213), Grade IV (175) | `REPORTED` |
| **Primary Repository** | Private GitHub: `https://github.com/asmkhalid111/glioma-fyp-research-hub` | `VERIFIED-CODE` |

### Clinical Problem & Scientific Motivation
Adult-type diffuse gliomas have historically been classified into histological grades (WHO Grades II, III, IV). Differentiating Grade II from Grade III gliomas based solely on histology is subject to marked interobserver variability (mitotic counting discordance, cellularity thresholds, Ki-67 scoring). Machine learning applied to bulk RNA-seq has been proposed to provide an objective, reproducible grading readout.

### The Modern Paradigm Shift: WHO CNS5 (2021)
The 2021 WHO Classification of Tumors of the Central Nervous System (WHO CNS5) fundamentally restructured glioma taxonomy. Gliomas are no longer graded across a single morphological continuum. Instead, grading is strictly performed **within molecularly defined types**:
1. *Astrocytoma, IDH-mutant* (Grades 2, 3, or 4; homozygous deletion of *CDKN2A/B* automatically confers Grade 4).
2. *Oligodendroglioma, IDH-mutant and 1p/19q-codeleted* (Grades 2 or 3).
3. *Glioblastoma, IDH-wildtype* (Grade 4 only; IDH-wildtype astrocytic gliomas with *TERT* promoter mutation, *EGFR* amplification, or $+7/−10$ chromosome copy number alterations are classified as Glioblastoma, WHO Grade 4).

Evaluating unstratified Grade II vs III vs IV classification based on legacy histology strings represents an obsolete study design.

---

## 2. Current Research Status & Implementation Progress

### Completed Stages:
- **Discovery (Prompt 1):** Completed. Documented in [incoming/01_research_project_understanding.md](incoming/01_research_project_understanding.md).
- **Independent Scientific Audit (Prompt 2):** Completed. Documented in [incoming/02_independent_scientific_audit.md](incoming/02_independent_scientific_audit.md).
- **Knowledge Base & Git Initialization:** Completed. Private repository initialized, verified, and synchronized with GitHub.

### Current Research Stage:
- **Phase 0 / Gate 0:** Pre-Execution Feasibility & Governance Gate.
- **Active Milestone:** **Milestone M0: Read-Only Data & Label Feasibility Audit**.
- **Model Training Status:** **HALTED / PROHIBITED** until Milestones M0 and M1 are formally approved.

### Implementation Progress (Observed in Phase-I Code):
- Loading and merging of TCGA-LGG and TCGA-GBM expression and clinical data (708 rows).
- Global variance filtering from 60,660 to 5,000 genes prior to splitting (leakage W3).
- Stratified 80/20 train/test split (566 train, 142 test), ungrouped by patient (W4).
- Synthetic oversampling via SMOTE on high-dimensional continuous gene space (W8, W9).
- Model benchmarks: Random Forest (78.17%), SVM (80.99%), Optuna-tuned RF (77.46%), XGBoost, DNN, Contrastive KNN.
- Saved Grade IV recall: ~0.97 across models; Grade II vs III subset accuracy: 62%–71% (`REPORTED`, unreproduced).
- Headline discrepancy (W12): Written Phase-I report claimed 84.51% RF accuracy with `log2(TPM + 1)` and MI+LASSO+RF feature selection; code shows raw TPM variance filtering and gets 78.17%. The 84.51% figure is unverified.

---

## 3. Scientific Context & Key Audit Findings

The independent scientific audit established the following foundational facts:

1. **The Problem is Saturated:** Over 85 machine learning studies for glioma grading exist in the literature (`REPORTED-SRC` S7). However, most evaluate MRI radiomics rather than RNA-seq. Reporting quality is poor (mean TRIPOD adherence 44.5%), and external validation is rare (5 of 85 studies).
2. **Severe Cohort–Grade Confounding (W1):**
   - In the merged dataset, Grade IV samples originate **100% from TCGA-GBM** (predominantly IDH-wildtype).
   - Grade II and III samples originate **100% from TCGA-LGG** (predominantly IDH-mutant).
   - Grade IV prediction is therefore confounded by cohort batch identity and IDH mutation status. The extent to which these factors explain the high Grade IV recall (~0.97) requires empirical investigation rather than assumption.
3. **Ceccarelli Pan-Glioma Evidence (`REPORTED-SRC` S10):** Ceccarelli et al. (Cell 2016, 1,122 gliomas) demonstrated that *IDH mutation status is the single dominant driver of transcriptome clustering across diffuse gliomas*.
4. **Modality Boundaries:**
   - The ~74% accuracy reported by S9 (Front Oncol 2021) evaluated **pathology images integrated with molecular data**, NOT bulk RNA-seq, and cannot serve as an RNA-seq ceiling.
   - Systematic review metrics from S7 evaluated predominantly **MRI radiomics**, not transcriptomics.
5. **Phase-I Methodological Weaknesses (W1–W18):**
   - *W1 (Critical):* Cohort-grade alignment.
   - *W2 (Critical):* Legacy histological labels; NOS and Mixed forced to Grade II.
   - *W3 (High):* Global preprocessing gene filtering before splitting (data leakage).
   - *W4 (High):* Ungrouped splits; potential patient aliquot leakage.
   - *W5 (High):* Unfiltered sample types (recurrent `-02` or normal `-11` samples potentially included).
   - *W6 (High):* Test-set reuse for model selection and manual ensemble weighting (`0.30/0.30/0.15/0.25`).
   - *W8 (High):* SMOTE followed by naive Keras `validation_split=0.1` validates on synthetic minority rows.
   - *W10 (High):* Circular cohort diagnostic (`cohort` derived from `Target_Grade`).

---

## 4. Research Decisions & Proposed Directions

| Category | Status / Decisions | Epistemic Status |
|---|---|---|
| **Approved Decisions** | 1. Zero-prior-knowledge discipline and strict evidence taxonomy.<br/>2. Preserve historical reports verbatim in repository.<br/>3. Halt all model training until M0 and M1 are approved.<br/>4. Require patient-grouped nested cross-validation ($5 \times 5$).<br/>5. Follow TRIPOD+AI reporting guidelines. | **ACTIVE / APPROVED** |
| **Proposed Research Framing** | A candidate synthesis combining:<br/>- *Option A (Framing):* Methodological Rigor and Confounding Study.<br/>- *Option B (Core Task):* Within-Type Transcriptomic Grading under WHO CNS5.<br/>- *Option C (Conditional):* Prognostic Transcriptomic Risk Score. | **PROPOSED** (Pending Supervisor Review at M1) |
| **Proposed Hypotheses** | - *H1:* Confounding degradation upon removing cohort/IDH artifacts.<br/>- *H2:* Discriminability of Grade II vs III within molecular types.<br/>- *H3:* Independent prognostic utility beyond clinical covariates. | **PROPOSED** (Candidate Hypotheses) |
| **Pending Supervisor Decisions** | 1. Historical baseline vs WHO CNS5-approximate primary label.<br/>2. Final scope of candidate objectives (Option A + B + C synthesis).<br/>3. Relegation of normal brain transcriptomes to supplementary check.<br/>4. Target publication venue and institutional capstone deadlines. | **PENDING (M1)** |
| **Rejected / Discarded Patterns** | 1. Hand-picked manual ensemble weights (`0.30/0.30/0.15/0.25`).<br/>2. Global preprocessing filtering before cross-validation splits.<br/>3. Single ungrouped 80/20 train/test split.<br/>4. Circular cohort diagnostics.<br/>5. TabFM / deep tabular foundation models as core contributions. | **PROHIBITED / DISCARDED** |
| **Unresolved Conflicts** | CONF-01 (Label Target), CONF-02 (Normal Samples), CONF-03 (Cox PH Centrality), CONF-04 (Paper Framing). | **DOCUMENTED IN DECISION LOG** |

---

## 5. Summary of Key Evidence & Class Distributions

### Reference Molecular Distribution (TCGA 638-Sample Subset, S12):
```
Molecular Subgroup                 Grade II    Grade III    Grade IV    Total
IDH-wildtype                             20           72         127      219
IDH-mutant, 1p/19q-codeleted             91           72           0      163
IDH-mutant, non-codeleted               127          112           8      247
Total                                   238          256         135      629
```
*Takeaway:* Within IDH-mutant subtypes, Grade II vs III sample sizes are reasonably balanced ($N \approx 160–250$ per type). In this historical subset, Grade IV was 94.1% IDH-wildtype ($127/135$).

### Evidence Status Definitions:
- `VERIFIED-CODE`: Directly observed in the exported Python script.
- `REPORTED-SRC`: Primary source identified and reported by the scientific audit.
- `REPORTED`: Stated in secondary documents; unverified independently.
- `UNVERIFIED`: Plausible lead requiring empirical verification.
- `PROPOSED`: Prospective design decision or hypothesis.

---

## 6. Current Next Action: Milestone M0

### Milestone M0: Read-Only Data & Label Feasibility Audit
- **Objective:** Run a non-invasive Python audit script on the raw Google Drive data to verify sample identity, scale, and clinical completeness without altering any data.
- **Specific Tasks:**
  1. Inspect file headers and sample value ranges of `star_tpm` to verify scale (raw TPM vs `log2(TPM + 1)`).
  2. Parse TCGA sample barcodes: extract 12-character patient identifiers (`bcr_patient_barcode`) to count unique biological patients.
  3. Audit sample type codes: tally primary solid tumors (`-01`), recurrent tumors (`-02`), and normal solid tissues (`-11`).
  4. Generate full cross-tabulation of `primary_diagnosis` $\times$ `tumor_grade` $\times$ project (`TCGA-LGG` vs `TCGA-GBM`).
  5. Audit survival fields: determine total events and completeness for Overall Survival (`OS`, `OS.time`) and Progression-Free Interval (`PFI`, `PFI.time`).
  6. Attempt to locate the source code or notebook yielding the Phase-I reported 84.51% RF accuracy.
- **Expected Deliverable:** `AUDIT_REPORT_M0.md` detailing exact patient counts, sample breakdowns, expression scale verification, and survival statistics.
- **Completion Criteria:** Unambiguous confirmation of unique patient count, scale, and sample types before initiating Milestone M1 (Supervisor Checkpoint).

---

## 7. Mandatory AI Working Rules (from AGENTS.md)

Any external AI model reviewing or assisting with this research must strictly comply with these rules:
1. **Zero-Prior-Knowledge Discipline:** Do not assume familiarity with the project beyond what is explicitly stated in this context pack and supplied repository files.
2. **Never Hallucinate Facts:** Never invent citations, sample counts, feature counts, metadata column names, or test scores. If missing, write `UNKNOWN` or `UNVERIFIED`.
3. **Respect Evidence Labels:** Treat `REPORTED`, `REPORTED-SRC`, and `UNVERIFIED` items strictly as investigational leads, never as established ground truth.
4. **No Silent Revisions:** Never silently "correct" scientific discrepancies, harmonize conflicting recommendations, or convert reported claims into verified facts.
5. **No Model Authority:** AI model agreement is **not proof**. Models provide independent advisory analysis, not scientific ground truth.
6. **Data Privacy:** Never upload raw patient-level data, credentials, or restricted datasets to external AI platforms.

---

## 8. Canonical Source Documents & Relative Links

For deeper investigation into specific aspects of the project, refer to the authoritative files in the GitHub repository:
- Master Operational Manual & Epistemic Rules: [AGENTS.md](AGENTS.md)
- Complete Technical Progress & Baseline: [CURRENT_STATE.md](CURRENT_STATE.md)
- Scientific Blueprint & Proposed Hypotheses: [RESEARCH_BLUEPRINT.md](RESEARCH_BLUEPRINT.md)
- Decision Log & Supervisor Checkpoints: [DECISION_LOG.md](DECISION_LOG.md)
- Evidence & Weakness Register (W1–W18): [EVIDENCE_REGISTER.md](EVIDENCE_REGISTER.md)
- Phased Research Roadmap (M0–M8): [MILESTONE_REGISTER.md](MILESTONE_REGISTER.md)
- Ethical Data Governance Protocol: [DATA_GOVERNANCE.md](DATA_GOVERNANCE.md)
- Historical Discovery Report: [docs/historical/01_research_project_understanding.md](docs/historical/01_research_project_understanding.md)
- Historical Scientific Audit Report: [docs/historical/02_independent_scientific_audit.md](docs/historical/02_independent_scientific_audit.md)
