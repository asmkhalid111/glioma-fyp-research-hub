# DECISION_LOG.md — Decision Records, Conflicts & Checkpoint Register

> **Document Status:** Active Governance Record  
> **Source Documents:** [incoming/01_research_project_understanding.md](incoming/01_research_project_understanding.md) (§5, §9), [incoming/02_independent_scientific_audit.md](incoming/02_independent_scientific_audit.md) (§9, §10, §14)  
> **Epistemic Traceability:** All logged decisions and conflicts originate strictly from foundational research records.

---

## 1. Foundational Decision Conflicts (Audit vs. Earlier Roadmap)

During the discovery and audit phases, four major methodological conflicts emerged between the earlier Phase II roadmap and the independent scientific audit report (`01_understanding` §5):

| Conflict ID | Strategic Topic | Earlier Assistant Roadmap | Independent Scientific Audit | Current Status & Action Required |
|---|---|---|---|---|
| **CONF-01** | **Target Label Definition** | Make WHO CNS5-aligned classification the sole primary target immediately. | Keep historical WHO 2007/2016 labels as a documented baseline; introduce CNS5-approximate labels only if clinical/molecular metadata supports it. | **PENDING SUPERVISOR CHECKPOINT (M1).** Depends on M0 audit of available IDH, 1p/19q, and CDKN2A/B metadata. |
| **CONF-02** | **Role of Normal Brain Samples** | Deprioritize normal brain transcriptomes (GTEx, recount-brain) completely. | Core only if system must handle unknown inputs; otherwise supplementary out-of-distribution check. GTEx-vs-TCGA batch effects dominate biological signal. | **PENDING SUPERVISOR CHECKPOINT (M1).** Default recommendation: relegated to supplementary sanity check. |
| **CONF-03** | **Centrality of Cox PH Survival Analysis** | Framed as the central, headline result of the paper. | Strictly conditional on survival event counts, follow-up duration, and proportional hazards assumptions. High risk of proxying age/IDH. | **RESOLVED IN PRINCIPLE.** Downgraded from headline to conditional objective, pending M0 survival event audit. |
| **CONF-04** | **Overarching Paper Framing** | Reframe primarily as a novel "prognostic grading signature" paper. | Frame as a "Methodological Rigor, Confounding Exposure & Within-Type Grading" study (Option A framing). | **PENDING SUPERVISOR CHECKPOINT (M1).** Candidate synthesis (Option A framing + Option B core + Option C conditional) remains **PROPOSED**, awaiting supervisor review. |

---

## 2. Component-Level Methodological Decisions (Module F Register)

Formal decisions regarding pipeline components evaluated in the scientific audit (`02_audit` §9.2):

| Component / Technique | Audit Verdict | Justification & Methodological Rationale | Target Replacement / Implementation |
|---|---|---|---|
| **Data Ingestion & Merge** | **RETAIN (with filters)** | Core loader functions are valid, but require upstream sample-type and patient deduplication. | Add barcode filtering: retain only primary solid tumors (`sample_type == '01'`) and 1 aliquot per patient. |
| **Historical Labels (G2/G3/G4)** | **RETAIN (as baseline)** | Necessary to benchmark directly against prior literature and quantify the confounding effect. | Retain as documented baseline; do not present as modern clinical ground truth. |
| **CNS5-Approximate Labels** | **ADD (proposed primary target)** | Modern standard for neuro-oncology; evaluates genuine biological grading within molecular types. | Derive from Ceccarelli molecular table (IDH, 1p/19q, CDKN2A/B) if available; pending M0 and M1. |
| **Global Gene Filtering** | **CORRECT** | Preprocessing leakage occurs when variance filtering is applied across all samples prior to split (W3). | Move all gene filtering (zero/low expression, variance selection) strictly inside training CV folds. |
| **Single 80/20 Partitioning** | **REMOVE** | High variance; single split creates severe test-set reuse across multiple experiments (W6). | Replace with $5 \times 5$ or $10 \times 10$ nested, stratified, patient-grouped cross-validation. |
| **SMOTE on Continuous Genes** | **CORRECT / COMPARE** | Synthetic interpolation in 5,000-dimensional gene space may generate non-biological transcriptomes; Keras validation slice bug (W8). | Apply SMOTE exclusively inside training folds, or replace with class-weighted loss functions; compare explicitly. |
| **Core Classifiers (RF, XGB, SVM, Elastic-Net)** | **RETAIN** | Classical tabular models are robust, interpretable, and appropriate for small-sample transcriptomics ($N \approx 700$). | Retain as the core model benchmark suite; add Elastic-Net logistic regression as standard sparse baseline. |
| **Deep Learning (DNN, Contrastive KNN)** | **RECONSIDER** | Full-batch contrastive encoder and custom DNNs add parameters without proven performance advantage on tabular bulk RNA-seq. | Retain only if they demonstrate statistically significant superiority under nested CV; otherwise relegate to supplement. |
| **Manual Ensemble Weights (`0.30/0.30/0.15/0.25`)** | **REMOVE** | Post-hoc manual tuning on the held-out test set invalidates reported generalization metrics (W6). | Permanently eliminate hand-picked weights. Use unweighted soft-voting or strictly nested stacking. |
| **Tabular Foundation Model (TabFM)** | **REMOVE** | High parameter complexity, zero verified performance gains on small biological cohorts ($N \approx 700$). | Remove from main paper scope. Retain code cell as exploratory archive only. |
| **SHAP Feature Selection** | **CORRECT** | Phase-I SHAP was computed on SMOTE data, hardcoded to class index `[1]`, and lacked cross-fold stability analysis (W13). | Compute SHAP out-of-fold; evaluate gene selection stability across folds; map Ensembl IDs to HGNC symbols. |
| **Circular Cohort Diagnostic** | **REMOVE / REBUILD** | Phase-I diagnostic derived `cohort` directly from `Target_Grade` (W10). | Rebuild diagnostic using true TCGA project identifiers (`TCGA-LGG` vs `TCGA-GBM`) and true IDH calls. |
| **Tumor-vs-Normal Objective** | **RECONSIDER** | Trivial discrimination dominated by GTEx-vs-TCGA tissue acquisition and batch artifacts. | Exclude from core paper objectives; retain only as supplementary out-of-distribution sanity check. |

---

## 3. Critical Questions Affecting Next Decisions

These 10 unresolved scientific questions must be addressed prior to finalizing the Phase II execution protocol (`01_understanding` §9, `02_audit` §10):

### Immediate Questions for Supervisor (Checkpoint M1)
1. **Target Definition Approval:** Has the supervisor formally approved retaining historical grades as a baseline while adopting WHO CNS5-aligned labels as the primary target?
2. **Project Scope & Objective Pruning:** Is the supervisor willing to prune the candidate objectives down to a focused scope (Option A framing + Option B within-type grading + Option C conditional survival), eliminating tumor-vs-normal detection from the core paper?
3. **Publication Deadlines & Targets:** Is a Q1 journal submission a strict hard requirement or an aspirational stretch goal, and what is the institutional capstone deadline?
4. **Team Roles & Compute Resources:** What is the team division of labor, and what dedicated GPU/cluster compute is available beyond free-tier Google Colab?
5. **Phase-I 84.51% Provenance:** Can the team locate the original notebook or git branch where the 84.51% Random Forest accuracy was obtained, or should it be officially retracted as unverified?

### Empirical Questions for Data Audit (Milestone M0)
6. **Clinical Metadata Fields:** What exact clinical columns exist in the raw TCGA TSVs beyond `primary_diagnosis` (e.g., `tumor_grade`, `ajcc_pathologic_stage`, `prior_malignancy`)?
7. **Patient Barcodes & Replicates:** How many unique biological patients exist within the 708 rows once `bcr_patient_barcode` (first 12 characters) is extracted?
8. **Sample Types Present:** How many rows carry sample type codes `-02` (recurrent), `-11` (normal solid tissue), or `-06` (metastatic)?
9. **Expression Value Scale:** Are the values in `star_tpm` raw TPM ($\ge 0$) or already log-transformed `log2(TPM + 1)` (W11)?
10. **Survival Data Completeness:** How many total deaths / overall survival events (`OS.time`, `OS == 1`) exist per histological and molecular subgroup?

---

## 4. Decision Log History

| Date | Log ID | Decision Description | Decided By | Supporting Evidence | Status |
|---|---|---|---|---|---|
| 2026-11-13 | `DEC-001` | Initialize repository with strict zero-prior-knowledge boundary and evidence taxonomy. | Repository Architect | Instructions & `02_audit` §0 | Active |
| 2026-11-13 | `DEC-002` | Preserve original historical records verbatim in `incoming/` and `docs/historical/`. | Repository Architect | Instructions Step 3, bitwise SHA256 match | Active |
| 2026-11-13 | `DEC-003` | Halt all model execution and training until Milestone M0 data audit is completed. | Repository Architect | `02_audit` §11, P0 recommendation | Active |
| 2026-11-13 | `DEC-004` | Propose Option A (Confounding & Rigor) as overall framing, Option B (Within-Type) as core task, Option C (Survival) as conditional. | Repository Architect (Candidate Proposal) | `02_audit` §9.3 synthesis proposal | **PROPOSED (Pending Supervisor Approval at M1)** |
