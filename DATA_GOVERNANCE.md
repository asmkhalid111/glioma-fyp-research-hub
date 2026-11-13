# DATA_GOVERNANCE.md — Research Data Governance & Ethics Protocol

> **Document Status:** Active Governance Protocol  
> **Applicability:** All contributors, pipelines, scripts, and AI agents interacting with this repository.  
> **Source Documents:** `01_research_project_understanding.md`, `02_independent_scientific_audit.md`

---

## 1. Ethical Principles & Epistemic Boundaries

This research involves secondary analysis of human genomic and clinical data derived from open-access scientific consortia. All data management and analytical workflows must adhere to:
1. **The Declaration of Helsinki** ethical principles for medical research involving human subjects.
2. **NIH Genomic Data Sharing (GDS) Policy** governing public access to The Cancer Genome Atlas (TCGA) resources.
3. **Strict Non-Re-identification Pledge:** Contributors and automated systems must never attempt to identify individual patient donors or link genomic data to external identity records.

---

## 2. Data Classification & Access Tiers

| Data Tier | Description | Source Datasets | Access & Storage Policy |
|---|---|---|---|
| **Tier 1: Open-Access Public Data** | Publicly accessible bulk RNA-seq expression profiles (`star_tpm`) and de-identified clinical metadata. | TCGA-LGG, TCGA-GBM (GDC / UCSC Xena public hub) | Open access. Storage restricted to secure local/cloud directories; **never committed to Git**. |
| **Tier 2: Controlled-Access Genomics** | Individual-level germline variants, raw read alignments (BAM/FASTQ), or restricted somatic mutation calls. | dbGaP-controlled TCGA files | **PROHIBITED** without active dbGaP authorization. Currently `UNVERIFIED` / absent from this project. |
| **Tier 3: External Validation Cohort** | Bulk RNA-seq and clinical records from non-TCGA cohorts. | Chinese Glioma Genome Atlas (CGGA `mRNAseq_693`, `mRNAseq_325`) | Proposed external resource. Subject to CGGA data use policies and terms. Access currently `UNVERIFIED`. |
| **Tier 4: Reference / Normal Brain Data** | Non-neoplastic human post-mortem brain transcriptomes. | GTEx, recount-brain | Open access for academic research. Acknowledged batch-effect limitations; proposed for supplementary checks only. |

---

## 3. Storage, Tracking & Repository Safety Rules

1. **Zero Data in Git:**  
   Raw genomic matrices (`star_tpm`), clinical TSVs, intermediate arrays, and heavy model weights must **NEVER** be committed or pushed to the GitHub repository.
   - All data paths are ignored via `.gitignore`.
   - Raw data must reside in external secure cloud storage (e.g., Google Drive) or a designated, untracked local `data/` directory.
2. **No Credentials or Secrets:**  
   Authentication tokens, API keys, private keys (`.pem`), or local credentials must never be written into script files or committed to Git.
3. **Deterministic Preprocessing:**  
   Processed datasets must be fully reproducible from raw files using documented, version-controlled scripts with fixed random seeds.

---

## 4. Patient De-identification & Barcode Hygiene

TCGA sample identifiers encode crucial biological and technical metadata that must be handled systematically:

```
TCGA-02-0001-01A-01D-0182-01
└─────┬────┘ └┬┘ └┬┘ └┬┘ └──┬──┘ └┬┘
      │       │   │   │     │    └─ Plate ID
      │       │   │   │     └────── Center ID
      │       │   │   └──────────── Portion / Analyte
      │       │   └──────────────── Vial (A, B, C...)
      │       └──────────────────── Sample Type:
      │                              01: Primary Solid Tumor
      │                              02: Recurrent Solid Tumor
      │                              06: Metastatic
      │                              11: Solid Tissue Normal
      └──────────────────────────── Patient Barcode (bcr_patient_barcode, 12 chars)
```

### Mandatory Rules for Data Ingestion:
- **Patient Grouping (W4):** All cross-validation partitions and evaluation splits must group by the 12-character patient barcode (`bcr_patient_barcode`), never by raw sample row IDs.
- **Sample Type Filtering (W5):** Only primary untreated solid tumors (`sample_type == '01'`) may enter the primary grading cohort. Recurrent (`-02`) and normal (`-11`) samples must be filtered out to prevent phenotypic contamination.
- **Single Aliquot Retention:** If a patient has multiple primary tumor aliquots, a deterministic deduplication rule must select exactly one aliquot prior to splitting.

---

## 5. External Cohort Governance (CGGA)

The Chinese Glioma Genome Atlas (CGGA) is proposed as an independent external validation dataset. Before ingestion:
1. Verify access terms, user registration requirements, and citation guidelines on `cgga.org.cn`.
2. Inspect sample annotations to ensure recurrent and secondary tumors are segregated from primary gliomas.
3. Verify that 1p/19q status in CGGA (which was predicted from expression via Hu et al. 2017) is not treated as circular ground truth against expression models.
4. Establish whether biological patient overlap exists between `mRNAseq_693` and `mRNAseq_325` prior to pooling.

