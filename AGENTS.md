# AGENTS.md — Agent Operating Manual & Epistemic Guidelines

This document governs the behavior, epistemic discipline, and operational boundaries of all AI agents and human contributors interacting with the **FYDP Glioma RNA-seq Research Repository**.

---

## 1. Epistemic Ground Rules

Every agent operating in this repository must strictly adhere to the following rules (`02_audit` §0):

1. **Strict Zero-Prior-Knowledge Assumption:** Do not assume prior familiarity with the project, its datasets, or its past execution runs beyond the explicit records stored in this repository.
2. **Never Hallucinate or Invent Facts:** Never invent citations, DOIs, patient/sample counts, feature counts, metadata column names, or benchmark performance metrics. If a detail is missing, write `UNKNOWN` or `UNVERIFIED`.
3. **Respect Evidence Status Labels:** Every claim, metric, and finding must carry an explicit status tag from the repository taxonomy (`VERIFIED-CODE`, `REPORTED-SRC`, `PARTIAL`, `REPORTED`, `UNVERIFIED`, `ASSUMED`, `PROPOSED`, `UNKNOWN`). Treat `REPORTED`, `REPORTED-SRC`, and `UNVERIFIED` items strictly as investigational leads, never as independently verified ground truth.
4. **No Silent Revisions:** Never silently "correct" scientific discrepancies, harmonize conflicting recommendations, or convert reported claims into verified facts. Any change in direction must explicitly document what is **kept**, **corrected**, **removed**, or **reconsidered**.
5. **Rigorous Citation Hygiene:** Verify author lists, publication years, titles, and DOIs against primary literature sources (e.g., PubMed, CrossRef) before incorporating any citation into manuscript drafts or blueprints. Initializing this repository does not constitute independent literature verification.

---

## 2. Hard Knowledge Boundaries & Prohibited Assumptions

As of repository initialization, agents have **not** been provided verified access to:
- Raw RNA-seq read counts or TPM matrices (`star_tpm`).
- Original GDC/TCGA or UCSC Xena clinical TSV files.
- Complete execution logs or `.ipynb` output cells from the Phase-I Colab notebook.
- Full Phase-I institutional project report and grading rubrics.
- Direct supervisor meeting records or verbal feedback.
- Chinese Glioma Genome Atlas (CGGA) downloaded files (`mRNAseq_693`, `mRNAseq_325`).
- Supplementary molecular data (Ceccarelli 2016 tables, copy number alteration files, TERT promoter calls).

**Prohibited Agent Behaviors:**
- Do **not** claim to have inspected raw data or clinical distributions.
- Do **not** assert that the 84.51% Phase-I RF accuracy is valid or reproducible.
- Do **not** state that cohort or IDH effects have already been proven to explain the Grade IV recall; state that Grade IV prediction is confounded by cohort and molecular differences, and that the contribution of these factors requires empirical investigation.
- Do **not** describe the ~74% result from one pathology-image and molecular-data study (S9) as a universal accuracy ceiling for RNA-seq grading.
- Do **not** generalize imaging-based systematic-review performance figures (S7) to RNA-seq studies.
- Do **not** treat historical WHO 2007/2016 histological grades (Grade II, III, IV) as modern clinical ground truth without explicitly addressing WHO CNS5 (2021) molecular grading standards.
- Do **not** claim that candidate hypotheses (H1–H3) or proposed directions (Option A + B + C) are approved or frozen; they are strictly `PROPOSED` pending supervisor approval.
- Do **not** state that external validation on CGGA has been completed or secured; CGGA is a proposed validation resource whose accessibility and suitability remain to be confirmed.
- Do **not** propose complex deep learning architectures (e.g., contrastive learning, TabFM, complex neural nets) as primary scientific novelties.

---

## 3. Discarded & Prohibited Methodological Patterns

The independent scientific audit identified critical methodological weaknesses (`02_audit` §6, §9.2). Agents must **never** re-introduce the following flawed patterns:

| Discarded / Flawed Pattern | Weakness ID | Rationale for Rejection | Prescribed Replacement |
|---|---|---|---|
| **Gene filtering on entire dataset before split** | W3 | Preprocessing data leakage; inflates cross-validation performance | Gene filters must be encapsulated strictly within training folds |
| **Ungrouped patient splits** | W4 | Potential cross-fold leakage if multiple aliquots/samples belong to one patient | Grouped splits by 12-character patient barcode (`bcr_patient_barcode`) |
| **Unfiltered sample types** | W5 | Recurrent (`-02`) and normal (`-11`) tissue falsely labeled as primary glioma | Filter to primary untreated solid tumors (`-01`) only |
| **Single 80/20 split & test-set reuse** | W6 | Multiple testing and post-hoc optimization on held-out test data | Nested, stratified, patient-grouped cross-validation ($5 \times 5$ or $10 \times 10$) |
| **Premature SHAP on entire training set** | W7 | Feature selection leakage within internal Optuna tuning loops | Out-of-fold feature selection; compute SHAP strictly per fold |
| **SMOTE followed by naive `validation_split`** | W8 | Keras `validation_split=0.1` slices synthetic minority rows at the end of the array | SMOTE applied exclusively within training folds; validate on real samples |
| **Manual ensemble weighting** | W6 | Weights `0.30/0.30/0.15/0.25` hand-picked post-hoc on test data | Unweighted soft-voting or strictly nested stacking |
| **Circular cohort diagnostic** | W10 | Derived `cohort` directly from `Target_Grade` (trivial tautology) | Rebuild diagnostic using true project identifiers (`TCGA-LGG` vs `TCGA-GBM`) |
| **TabFM as core contribution** | W14 | Deep tabular foundation model on $N \approx 700$ adds complexity without utility | De-prioritize to supplementary exploration or remove entirely |

---

## 4. Operational Workflow for Future Agents

When a new agent session starts, follow this protocol:

### Step 1: Ingest Context
1. Read [AI_ONBOARDING_PROMPT.md](AI_ONBOARDING_PROMPT.md) for the concise executive summary.
2. Read [AI_CONTEXT_PACK.md](AI_CONTEXT_PACK.md) for the latest portable summary of project state, evidence, and open decisions.
3. Review [CURRENT_STATE.md](CURRENT_STATE.md) for technical progress and active blockers.
4. Review [DECISION_LOG.md](DECISION_LOG.md) to check which decisions remain open or pending supervisor approval.

### Step 2: Respect Project Gates
- **Gate 0 (Current State):** No model training or algorithmic development may occur until Milestone M0 (Read-Only Data Audit) is completed.
- **Gate 1:** Analysis plan and primary endpoints must be pre-registered ([RESEARCH_BLUEPRINT.md](RESEARCH_BLUEPRINT.md)) before external validation on CGGA data is conducted.

### Step 3: Modifying Knowledge Base Files
- Whenever a new experiment or audit is performed, update [CURRENT_STATE.md](CURRENT_STATE.md), [EVIDENCE_REGISTER.md](EVIDENCE_REGISTER.md), and [MILESTONE_REGISTER.md](MILESTONE_REGISTER.md).
- When an open tension or question is decided (e.g. by supervisor directive), record the decision in [DECISION_LOG.md](DECISION_LOG.md) with timestamp, rationale, alternatives considered, and impact.
- Maintain repository-relative markdown links for all document cross-references.

---

## 5. Standard Evaluation Standards (Mandatory for Future Runs)

All prospective modeling must report:
1. **Primary Metrics:** Balanced Accuracy, Macro-Averaged F1-Score, Matthews Correlation Coefficient (MCC), and Cohen's Kappa ($\kappa$). Overall accuracy must **not** serve as the headline metric due to class imbalance (`02_audit` §8, W16).
2. **Per-Class Metrics:** Recall (Sensitivity), Specificity, Precision, and Area Under ROC/PRC per grade.
3. **Statistical Uncertainty:** 95% Bootstrap Confidence Intervals ($B \ge 1,000$) or repeated nested CV variance across folds.
4. **Baselines:** Always compare against simple elastic-net logistic regression and a clinical/molecular baseline (Age + IDH + 1p/19q alone) to quantify the marginal value of RNA-seq features.
5. **Compliance:** Follow the **TRIPOD+AI** (Transparent Reporting of a multivariable prediction model of Individual Prognosis Or Diagnosis) reporting checklist for all publication outputs.

---

## 6. Multi-Model Collaboration & AI Review Rules

When external AI models (via T3 Chat, Claude, ChatGPT, Gemini, etc.) are consulted:
1. **Advisory Role Only:** External AI models provide independent advisory opinions, intellectual stress-testing, and peer critiques. They possess **zero authority** to approve hypotheses, finalize methodology, or modify canonical repository records.
2. **Model Agreement is Not Proof:** Multiple AI models agreeing on a point does not constitute scientific evidence. Models may share common training-data heuristics or literature biases.
3. **Canonical Authority:** The GitHub repository `glioma-fyp-research-hub` is the sole source of truth. Models must review materials grounded in [AI_CONTEXT_PACK.md](AI_CONTEXT_PACK.md) pinned to a specific Git commit SHA.
4. **No Silent Revisions:** Recommendations originating from AI reviews must be documented in [docs/ai_reviews/syntheses/](docs/ai_reviews/syntheses/) and formally submitted to the research supervisor before any change is made to canonical files.
5. **Review Preservation:** All raw external model responses must be preserved in [docs/ai_reviews/incoming/](docs/ai_reviews/incoming/) and archived using [docs/templates/AI_REVIEW_RECORD_TEMPLATE.md](docs/templates/AI_REVIEW_RECORD_TEMPLATE.md).
6. **Context Pack Maintenance:** [AI_CONTEXT_PACK.md](AI_CONTEXT_PACK.md) must be refreshed following the protocol in [docs/templates/CONTEXT_PACK_UPDATE_PROCEDURE.md](docs/templates/CONTEXT_PACK_UPDATE_PROCEDURE.md) whenever milestones or decisions change.
