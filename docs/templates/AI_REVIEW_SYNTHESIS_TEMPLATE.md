# Multi-Model AI Review Synthesis Template

> **Template Status:** Standard Research Archive Template  
> **Usage:** Use this template to compile structured cross-model synthesis reports in `docs/ai_reviews/syntheses/`.  
> **Governance Notice:** Agreement among models is **not proof**. Consensus does not grant automatic approval. Final decisions require human evaluation and supervisor authorization.

---

## 1. Synthesis Metadata & Evaluated Models

| Metadata Attribute | Synthesis Record Value |
|---|---|
| **Synthesis ID** | `SYN-[TASK_ID]-[TOPIC]-[YYYYMMDD]` (e.g., `SYN-M01-CGGA-FEASIBILITY-20261114`) |
| **Synthesis Date** | YYYY-MM-DD |
| **Research Task / Question** | [State the exact research question evaluated across models] |
| **Synthesizing Agent / Human** | [Name of human researcher or synthesis agent] |
| **AI Context Pack Version** | e.g., v1.0.0 |
| **Base Git Commit SHA** | [Full 40-char SHA of repo when synthesis was prepared] |
| **Target Milestone** | e.g., Milestone M0 / Milestone M1 / Milestone M4 |

### Models Consulted:
1. **Model A:** [Provider / Model Name / Role Assigned / Review ID]
2. **Model B:** [Provider / Model Name / Role Assigned / Review ID]
3. **Model C:** [Provider / Model Name / Role Assigned / Review ID]

---

## 2. Executive Synthesis & Bottom-Line Assessment
Provide a concise executive overview comparing the findings of the consulted models. Highlight whether a clear scientific direction emerged or whether deep conceptual divides exist.

---

## 3. Areas of Independent Agreement (Consensus Analysis)
Detail findings, critiques, or recommendations where multiple models independently converged:
- **Consensus Area 1:** [Detail topic] — *Models concurring:* [List models]. *Scientific basis:* [Primary literature or mathematical logic supporting consensus].
- **Consensus Area 2:** [Detail topic] — *Models concurring:* [List models]. *Scientific basis:* [Primary literature or mathematical logic supporting consensus].

> *Critical Check:* Did models agree because of shared training set heuristics, or because of robust biological/methodological reality?

---

## 4. Material Disagreements & Divergent Interpretations
Detail points where models reached conflicting conclusions or offered contradictory recommendations:
- **Divergence 1:**  
  - *Model A Position:* [State position and rationale]  
  - *Model B Position:* [State position and rationale]  
  - *Analytical Root Cause:* [Why did they diverge? e.g., different risk tolerance, differing clinical priorities, divergent interpretations of WHO CNS5].

---

## 5. Claims Requiring Primary Source Verification
List factual statements, citations, or statistical claims made by models that require independent verification against primary data or literature:
- [ ] Claim 1: [State claim and model source] $\rightarrow$ [Verification outcome]
- [ ] Claim 2: [State claim and model source] $\rightarrow$ [Verification outcome]

---

## 6. Unsupported, Questionable, or Hallucinated Claims
Explicitly list any claims made by models that were found to be unsubstantiated, methodologically flawed, or factually inaccurate upon human audit:
- *Identified Flaw 1:* [Describe claim, model, and why it was rejected].

---

## 7. Actionable Synthesis Recommendations
Provide a synthesized, prioritized list of recommendations for the research project:
1. **P1 (Immediate Action):** [Recommendation]
2. **P2 (Design Adjustment):** [Recommendation]
3. **P3 (Exploratory / Deferred):** [Recommendation]

---

## 8. Decisions Requiring Human & Supervisor Approval
Formulate the exact decision items that must be submitted to the project supervisor at the next checkpoint:
- **Decision Item 1:** [Exact question and proposed recommendation]
- **Decision Item 2:** [Exact question and proposed recommendation]

---

## 9. Final Disposition & Canonical Status
- **Current Disposition:** `PENDING SUPERVISOR REVIEW` / `ACCEPTED WITH MODIFICATIONS` / `REJECTED`
- **Supervisor Action Taken:** [Record date, feedback, and supervisor directive]
- **Repository Updates Made:** [List canonical documents updated, e.g., DECISION_LOG.md `DEC-XXX`]
