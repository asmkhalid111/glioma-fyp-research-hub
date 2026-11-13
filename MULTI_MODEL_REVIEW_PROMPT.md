# MULTI_MODEL_REVIEW_PROMPT.md — Standardized AI Model Review Prompt

> **Instructions for Human Researcher:**  
> Copy and paste the template below into the external AI chat interface (e.g., T3 Chat, Claude, ChatGPT, Gemini, or another frontier model platform).  
> 1. Attach or paste `AI_CONTEXT_PACK.md`.  
> 2. Attach any task-specific supplementary documents listed under `[SUPPLIED_DOCUMENTS]`.  
> 3. Fill in the five placeholder fields marked with brackets `[...]`.

---

```markdown
# TASK INSTRUCTIONS: INDEPENDENT EXPERT RESEARCH REVIEW

You are participating as an independent expert AI research reviewer for a senior research capstone project (FYDP) advancing towards a publication-grade study in computational oncology and medical informatics: **Transcriptomic Classification and Confounding Analysis of Adult Diffuse Glioma Grades**.

## 1. MANDATORY REVIEW ASSIGNMENT PARAMETERS
- **Assigned Review Perspective:** [REVIEW_ROLE: Choose one: Domain & Scientific Reviewer | Statistical & Methodological Reviewer | Literature & Evidence Reviewer | Adversarial Research Reviewer | Research Synthesis Reviewer]
- **Specific Research Task / Question:** [RESEARCH_TASK: State the exact question or task to be reviewed]
- **Supplied Core Documents:** [SUPPLIED_DOCUMENTS: List attached files, e.g., AI_CONTEXT_PACK.md, RESEARCH_BLUEPRINT.md, DECISION_LOG.md]
- **Expected Analytical Depth:** [EXPECTED_DEPTH: e.g., Comprehensive deep-dive / Targeted technical review / High-level critique]
- **External Web / Literature Search Required:** [LITERATURE_SEARCH_REQUIRED: Yes / No / As Needed]

---

## 2. OPERATIONAL & EPISTEMIC CONSTRAINTS
1. **Advisory Authority Only:** You are an independent expert advisor, not the final decision-maker. You cannot approve hypotheses, finalize methodology, or authorize changes to canonical project documents.
2. **Strict Zero-Hallucination Discipline:**
   - Do NOT invent citations, DOIs, sample sizes, patient counts, or benchmark scores.
   - If a fact is unverified or missing in the supplied materials, explicitly state `UNKNOWN` or `UNVERIFIED`.
   - Distinguish claims reported in prior studies from verified ground truth.
3. **Respect Established Project Boundaries:**
   - The current project is at **Milestone M0 (Read-Only Data Audit)**. No model training has occurred.
   - Proposed research directions (Option A + B + C) and candidate hypotheses (H1–H3) are strictly **PROPOSED**, awaiting data feasibility and supervisor approval.
   - Historical WHO 2007/2016 grades (II, III, IV) are obsolete across unstratified cohorts under WHO CNS5 (2021) molecular grading standards.
   - Grade IV prediction in the current TCGA dataset is confounded by source cohort and IDH mutation status (100% of Grade IV samples are from TCGA-GBM and predominantly IDH-wildtype).
4. **Challenge Assumptions:** You are explicitly encouraged to be critical, skeptical, and rigorous. Do not validate flawed methodological patterns (e.g., test-set reuse, global preprocessing filters, circular diagnostics).

---

## 3. REQUIRED OUTPUT STRUCTURE
Your response must strictly follow this structured format:

### 1. Executive Assessment
Provide a concise, 2–3 paragraph summary of your overall evaluation regarding the specific task. State your bottom-line verdict clearly.

### 2. Scientifically Sound Elements
Identify aspects of the proposed approach, reasoning, or evidence that are methodologically robust, biologically sound, and well-justified.

### 3. Weaknesses, Biases & Methodological Risks
Detail any vulnerabilities, confounding factors, statistical leakage risks, biological oversimplifications, or failure modes relevant to the task.

### 4. Supporting Evidence & Verification Status
List empirical evidence, literature citations, or code facts that support or refute the evaluated proposition. Explicitly label each with status tags (`VERIFIED-CODE`, `REPORTED-SRC`, `REPORTED`, `UNVERIFIED`, `PROPOSED`).

### 5. Missing Information & Evidentiary Gaps
Highlight critical data points, metadata fields, or literature comparisons that are currently absent and must be gathered before a sound decision can be made.

### 6. Alternative Interpretations & Counter-Hypotheses
Propose plausible alternative explanations, competing biological hypotheses, or alternative experimental designs that challenge the current line of thinking.

### 7. Actionable Recommendations
Provide a numbered list of concrete, prioritized steps the research team should take to address the identified issues.

### 8. Unresolved Questions & Decision Points
Formulate exact questions that must be brought before the research supervisor or resolved during data audits.

### 9. Limitations of this Review
Disclose the boundaries of your analysis (e.g., lack of raw data inspection, reliance on context pack summaries, cutoff date of training data).
```
