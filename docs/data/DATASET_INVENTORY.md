# DATASET_INVENTORY.md — Genomic & Clinical Dataset Inventory

> **Document Status:** Reference Catalog  
> **Source Documents:** `01_research_project_understanding.md`, `02_independent_scientific_audit.md`  
> **Storage Notice:** Raw data files reside on external secure cloud storage (Google Drive) and are never committed to this repository.

---

## 1. Primary Dataset: TCGA Pan-Glioma (LGG & GBM)

| Attribute | Specification | Evidence Status |
|---|---|---|
| **Cohorts** | TCGA-LGG (Brain Lower Grade Glioma) & TCGA-GBM (Glioblastoma Multiforme) | `VERIFIED-CODE` (referenced in script) |
| **Data Types** | Bulk RNA-seq (`star_tpm`) and clinical metadata TSVs | `VERIFIED-CODE` |
| **Merged Row Count** | 708 rows total (Grade II: 320, Grade III: 213, Grade IV: 175) | `REPORTED` |
| **Unique Patients** | Barcode trimming to 12 chars required; duplicate count currently `UNVERIFIED` | `UNVERIFIED` (W4) |
| **Expression Units** | STAR-aligned TPM matrices; log2 scaling vs raw TPM currently `UNVERIFIED` | `UNVERIFIED` (W11) |
| **Molecular Metadata** | Ceccarelli et al. (Cell 2016) matched annotations (IDH, 1p/19q, subtype) | `REPORTED-SRC` (S10, S11) |

---

## 2. Proposed External Validation Dataset: CGGA

| Attribute | Specification | Evidence Status |
|---|---|---|
| **Resource** | Chinese Glioma Genome Atlas (cgga.org.cn) | `REPORTED-SRC` (S5, S6) |
| **Datasets** | `mRNAseq_693` (693 samples) and `mRNAseq_325` (325 samples) | `REPORTED-SRC` (S5, S6) |
| **Sequencing Platform** | Illumina HiSeq, STAR + RSEM pipeline | `REPORTED-SRC` (S5, S6) |
| **Tumor Types** | Includes Primary LGG, Recurrent LGG, Primary GBM, Recurrent GBM, Secondary GBM | `REPORTED-SRC` (S5, S6) |
| **Filtering Requirement** | Must filter strictly to primary untreated gliomas for unbiased validation | `PROPOSED` |
| **IDH & 1p/19q Calls** | IDH inferred from sequencing reads; 1p/19q predicted from expression (Hu et al.) | `REPORTED-SRC` (S5) |
| **Current Project Status** | Proposed external validation resource; data access, overlap, and pipeline compatibility currently `UNVERIFIED` | `UNVERIFIED` |

---

## 3. Reference Cohorts: Normal Brain (GTEx / recount-brain)

| Attribute | Specification | Evidence Status |
|---|---|---|
| **Resource** | Genotype-Tissue Expression (GTEx) / recount-brain | `REPORTED` |
| **Tissue Type** | Non-neoplastic post-mortem human brain tissue | `REPORTED` |
| **Methodological Concern** | GTEx vs TCGA technical batch effects dominate biological signals | `PROPOSED` |
| **Current Project Status** | Candidate resource; relegated to supplementary out-of-distribution sanity check | `PROPOSED` |

