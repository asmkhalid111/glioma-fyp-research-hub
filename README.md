# FYDP Glioma RNA-seq Research Repository

> **Epistemic Status Notice & Knowledge Boundary**  
> This repository was initialized on **13 November 2026** exclusively from two foundational research records:
> 1. `01_research_project_understanding.md` (Discovery / Understanding Report)
> 2. `02_independent_scientific_audit.md` (Independent Scientific Audit & Literature Evaluation)
> 
> **Crucial Boundary:** Antigravity and any downstream AI agents have **not** inspected raw RNA-seq data, original clinical TSVs, complete Jupyter notebook outputs, full Phase-I experiment logs, direct supervisor communications, CGGA datasets, or additional molecular mutation files. All entries herein are strictly traceable to the two supplied documents. Nothing has been independently reverified against primary raw data or computational runs. Literature sources (S1–S13) were identified by the audit report and have not been independently inspected by Antigravity during repository initialization.

---

## 1. Project Overview

| Attribute | Status / Value | Source / Evidence |
|---|---|---|
| **Project Identity** | Transcriptomic classification of diffuse glioma grades (WHO Grade II, III, IV) using RNA-seq | `01_understanding` §1, `02_audit` §1 |
| **Domain** | Computational Oncology, Transcriptomics, Machine Learning | REPORTED |
| **Primary Datasets** | TCGA-LGG (Lower Grade Glioma) & TCGA-GBM (Glioblastoma Multiforme), STAR-TPM | VERIFIED-CODE (referenced in `.py`) |
| **Current Sample Size** | 708 merged samples: Grade II (320), Grade III (213), Grade IV (175) | REPORTED |
| **Current Stage** | Pre-execution Phase II: Independent Audit Complete. Awaiting P0 Data Feasibility Audit & Supervisor Checkpoint | `02_audit` §11, §14 |
| **Target Venue** | Aiming towards a Q1 Computational Oncology / Medical Informatics journal (stretch goal) | REPORTED / UNVERIFIED |

### Problem Statement & Clinical Motivation
Diffuse gliomas are primary central nervous system neoplasms historically categorized into histological grades (WHO Grade II, III, IV). Distinguishing Grade II from Grade III based solely on histology suffers from well-documented subjective interobserver variability (mitotic counting, cellularity, Ki-67 thresholds). Machine learning on bulk RNA-seq has been proposed to provide an objective, reproducible grading readout.

However, the field has undergone a profound paradigm shift: the **2021 WHO Classification of Tumors of the Central Nervous System (WHO CNS5)** integrated molecular markers (IDH mutation, 1p/19q codeletion, CDKN2A/B homozygous deletion, TERT promoter mutation, EGFR amplification, +7/−10 copy number alterations) directly into tumor definitions. Grading is now performed *within* molecular types rather than across all gliomas as a single continuum.

---

## 2. Executive Scientific Verdict (Audit Summary)

Based on the independent scientific audit (`02_audit` §2):
1. **The problem is real but not novel:** TCGA glioma grading using classical ML is saturated. Over 85 studies exist (mostly MRI radiomics, some transcriptomics), frequently suffering from poor reporting (mean TRIPOD adherence 44.5%) and lack of external validation.
2. **The target label is the critical scientific vulnerability:** The current Grade II/III/IV targets are derived from legacy WHO 2007/2016 histological strings (`primary_diagnosis.diagnoses`). Histological Grade II/III/IV across unstratified cohorts is obsolete under WHO CNS5. It may serve as a historical baseline, but cannot form the primary scientific claim of a modern Q1 paper.
3. **Cohort–Grade Confounding (W1):** In the existing dataset, Grade IV samples originate **100% from TCGA-GBM** (predominantly IDH-wildtype), while Grades II and III originate **100% from TCGA-LGG** (predominantly IDH-mutant). Grade IV prediction is thus confounded by cohort and molecular differences; the exact extent to which these factors account for the high Grade IV recall (~0.97) requires empirical investigation.
4. **Current Phase-I Metrics are Compromised:** Reported accuracies (SVM 80.99%, RF 78.17%) suffer from preprocessing gene selection leakage (W3), test-set reuse for model selection and ensemble weight tuning (W6), lack of patient-level grouping (W4), and a SMOTE/validation slice artifact (W8).
5. **Phase-I Headline Claim Discrepancy (W12):** The Phase-I report claimed **84.51%** accuracy with log2(TPM+1), MI+LASSO+RF, and top-500 genes on an 80/10/10 split. The actual codebase shows no log transform, variance filtering to 5,000 genes, an 80/20 split, and achieves 78.17%. The 84.51% figure is **not reproduced**.
6. **Proposed Strategic Direction:** Shift away from an incremental "best accuracy" benchmark. A candidate research direction combining Option A (Methodological Rigor and Confounding Study framing), Option B (Within-Type Grading as core task), and Option C (Prognostic Transcriptomic Score as conditional add-on) has been proposed, with external validation proposed on CGGA primary tumors. This framing remains **PROPOSED** and is strictly pending data feasibility (Milestone M0) and supervisor approval (Milestone M1).

---

## 3. Evidence Classification Taxonomy

All statements, data points, and hypotheses in this repository strictly adhere to this evidence hierarchy (`02_audit` §0):

| Label | Definition | Usage in Repository |
|---|---|---|
| `VERIFIED-CODE` | Directly observed and verified in the supplied Colab `.py` script | Exact lines of code, preprocessing steps, model parameters |
| `REPORTED-SRC` | Supported by an external primary source identified and reported in the audit | Peer-reviewed citations, WHO CNS5 guidelines, CGGA portal facts |
| `PARTIAL` | Literature or evidence supports only a portion of the stated claim | Claims partially confirmed but missing specific parameters |
| `REPORTED` | Stated by the user or in secondary documents not directly accessible | Sample counts, Phase-I report metrics, supervisor comments |
| `UNVERIFIED` | Plausible hypothesis or lead lacking direct empirical confirmation | Gene annotations, overlap between cohorts, column names |
| `ASSUMED` | Working baseline assumption requiring formal validation | Independence of rows, comparability of TPM values |
| `PROPOSED` | Methodological recommendation or prospective design decision | Proposed nested CV architecture, candidate hypotheses, proposed options |
| `UNKNOWN` | Information completely absent from available records | Team size, GPU budget, raw clinical TSV column lists |

---

## 4. Multi-Model AI Collaboration System

To enable independent scientific peer critique without compromising data safety or repository integrity, this repository includes a multi-model AI collaboration infrastructure:

- **Portable Research Context:** [AI_CONTEXT_PACK.md](AI_CONTEXT_PACK.md) provides a compact, self-contained synthesis of the latest project state that can be supplied to external AI models (e.g. via T3 Chat, Claude, ChatGPT, Gemini).
- **Collaboration Protocol:** [MULTI_MODEL_REVIEW_PROTOCOL.md](MULTI_MODEL_REVIEW_PROTOCOL.md) formalizes five independent review perspectives (Domain, Methodological, Statistical, Literature, and Adversarial) and enforces a strict evidence hierarchy (AI models provide advisory critiques, not ground truth).
- **Standardized Review Prompt:** [MULTI_MODEL_REVIEW_PROMPT.md](MULTI_MODEL_REVIEW_PROMPT.md) provides a universal, model-agnostic prompt template to ensure consistent evaluation across different frontier AI systems.
- **Review Archive:** All incoming and vetted model reviews are tracked and archived in [docs/ai_reviews/](docs/ai_reviews/README.md) with structured templates for individual reviews ([AI_REVIEW_RECORD_TEMPLATE.md](docs/templates/AI_REVIEW_RECORD_TEMPLATE.md)) and multi-model syntheses ([AI_REVIEW_SYNTHESIS_TEMPLATE.md](docs/templates/AI_REVIEW_SYNTHESIS_TEMPLATE.md)).

---

## 5. Repository Navigation & Structure

```
FYDP-Research/
├── README.md                              # Repository overview, epistemic status, and navigation (this file)
├── AGENTS.md                              # Operating manual and epistemic discipline for autonomous agents
├── AI_CONTEXT_PACK.md                     # Portable research context pack for external AI models
├── AI_ONBOARDING_PROMPT.md                # Quick-start context prompt for new AI sessions
├── CURRENT_STATE.md                       # Comprehensive synthesis of current technical and empirical state
├── DATA_GOVERNANCE.md                     # Data tiers, ethics, patient de-identification, and security protocol
├── DECISION_LOG.md                        # Record of decisions, open conflicts, and pending supervisor checkpoints
├── EVIDENCE_REGISTER.md                   # Catalog of code findings, literature (S1-S13), weaknesses (W1-W18)
├── MILESTONE_REGISTER.md                  # Research roadmap from M0 (Feasibility) to M8 (Manuscript)
├── MULTI_MODEL_REVIEW_PROTOCOL.md         # Multi-model review roles, procedures, and evidence hierarchy
├── MULTI_MODEL_REVIEW_PROMPT.md           # Universal model-independent review prompt template
├── RESEARCH_BLUEPRINT.md                  # Scientific foundation, hypotheses, confounding analysis, and study design
├── .gitignore                             # Git ignore rules protecting raw data, checkpoints, and secrets
├── incoming/                              # Original historical input files (unabridged raw records)
│   ├── 01_research_project_understanding.md
│   └── 02_independent_scientific_audit.md
└── docs/
    ├── ai_reviews/                        # Multi-model review archive and collaboration hub
    │   ├── README.md                      # Archive guidelines and T3 Chat workflow
    │   ├── incoming/                      # Raw, unvetted external AI responses
    │   ├── independent_reviews/           # Vetted reviews categorized by role (scientific, methodological, etc.)
    │   └── syntheses/                     # Cross-model synthesis and reconciliation reports
    ├── historical/                        # Exact bitwise preserved copies of initial foundational reports
    │   ├── 01_research_project_understanding.md
    │   └── 02_independent_scientific_audit.md
    ├── 01_discovery/                      # Preserved discovery report copy
    │   └── 01_research_project_understanding.md
    ├── 02_scientific_audit/               # Preserved scientific audit report copy
    │   └── 02_independent_scientific_audit.md
    ├── templates/                         # Research governance and review templates
    │   ├── ANALYSIS_PLAN_TEMPLATE.md
    │   ├── AI_REVIEW_RECORD_TEMPLATE.md
    │   ├── AI_REVIEW_SYNTHESIS_TEMPLATE.md
    │   └── CONTEXT_PACK_UPDATE_PROCEDURE.md
    └── data/                              # Genomic and clinical data documentation
        └── DATASET_INVENTORY.md
```

### Document Cross-References (Repository-Relative Links):
- Operational guidelines for agents: [AGENTS.md](AGENTS.md)
- Portable research context pack: [AI_CONTEXT_PACK.md](AI_CONTEXT_PACK.md)
- Context prompt for new AI sessions: [AI_ONBOARDING_PROMPT.md](AI_ONBOARDING_PROMPT.md)
- Verified baseline and state of the codebase: [CURRENT_STATE.md](CURRENT_STATE.md)
- Scientific methodology and proposed hypotheses: [RESEARCH_BLUEPRINT.md](RESEARCH_BLUEPRINT.md)
- Decision log and pending supervisor checkpoints: [DECISION_LOG.md](DECISION_LOG.md)
- Master evidence and literature register: [EVIDENCE_REGISTER.md](EVIDENCE_REGISTER.md)
- Milestone roadmap and execution gates: [MILESTONE_REGISTER.md](MILESTONE_REGISTER.md)
- Multi-model collaboration protocol: [MULTI_MODEL_REVIEW_PROTOCOL.md](MULTI_MODEL_REVIEW_PROTOCOL.md)
- Reusable review prompt template: [MULTI_MODEL_REVIEW_PROMPT.md](MULTI_MODEL_REVIEW_PROMPT.md)
- Data governance and ethics protocol: [DATA_GOVERNANCE.md](DATA_GOVERNANCE.md)

---

## 6. Immediate Next Steps (Priority P0)

Before executing any machine learning pipelines or modifying models:
1. **Resolve Critical Questions:** Supervisor alignment on historical vs CNS5-approximate target labels, project scope (core vs optional objectives), and publication timeline (`01_understanding` §9).
2. **Execute Milestone M0 (Read-Only Data Audit):** Run a non-invasive Python script on the Google Drive raw files to verify:
   - Unique patient barcodes (`bcr_patient_barcode`).
   - Sample type codes (filter out recurrent `-02` and normal `-11` samples; retain `-01` primary tumors only).
   - Cross-tabulation of `primary_diagnosis` × `tumor_grade` × project (`TCGA-LGG` vs `TCGA-GBM`).
   - Expression scale check (verify whether `star_tpm` values are raw TPM or `log2(TPM + 1)`).
   - Survival field completeness and event counts (`OS`, `OS.time`, `PFI`).
3. **Trace the 84.51% Source:** Identify whether the Phase-I reported result stems from an alternative notebook or uncommitted code branch (`01_understanding` §3, §9).
