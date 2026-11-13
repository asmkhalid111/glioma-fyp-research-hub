# Independent AI Review Record Template

> **Template Status:** Standard Research Archive Template  
> **Usage:** Use this template to file and preserve every external AI model review in `docs/ai_reviews/independent_reviews/<category>/`.

---

## 1. Review Metadata & Provenance

| Metadata Attribute | Review Record Value |
|---|---|
| **Review ID** | `REV-[TASK_ID]-[ROLE]-[PROVIDER]-[YYYYMMDD]` (e.g., `REV-M01-METHOD-ANTHROPIC-20261114`) |
| **Execution Date** | YYYY-MM-DD |
| **AI Platform / Service** | e.g., T3 Chat / Claude Web / ChatGPT / Google AI Studio / Direct API |
| **Model Provider** | e.g., Anthropic / OpenAI / Google / Meta / Mistral |
| **Exact Model Name & Version** | e.g., Claude 3.5 Sonnet (20241022) / GPT-4o (2024-08-06) / Gemini 1.5 Pro |
| **Assigned Review Role** | Domain & Scientific / Methodological / Statistical / Literature / Adversarial |
| **Research Task / Question** | [State the precise research question evaluated] |
| **AI Context Pack Version** | e.g., v1.0.0 |
| **Base Git Commit SHA** | [Full 40-char SHA of repo when review was generated] |
| **Supplied Documents** | [List all files attached or pasted into the prompt] |
| **Web Search Enabled** | Yes / No / Unknown |
| **Verification Status** | Unaudited / Partially Verified / Verified against Primary Sources |

---

## 2. Review Execution Parameters & Prompt Record
- **System Prompt Used:** [Reference to MULTI_MODEL_REVIEW_PROMPT.md or custom variant]
- **Specific Prompt Customizations:** [Document any task-specific additions to the prompt]

---

## 3. Complete Model Response (Unabridged)

> *Paste the complete, unaltered text of the AI model's response below:*

```markdown
[PASTE COMPLETE AI MODEL OUTPUT HERE]
```

---

## 4. Human Verification & Critical Audit

*This section must be completed by the human researcher before filing:*

### 4.1 Claim-by-Claim Verification:
- **Claim 1:** [State claim from review] $\rightarrow$ **Status:** `VERIFIED-SRC` / `UNVERIFIED` / `REFUTED` (Citing: [Source])
- **Claim 2:** [State claim from review] $\rightarrow$ **Status:** `VERIFIED-SRC` / `UNVERIFIED` / `REFUTED` (Citing: [Source])

### 4.2 Hallucination & Fact-Checking Audit:
- *Did the model invent any citations or DOIs?* [Yes/No — Detail if any]
- *Did the model misinterpret project boundaries?* [Yes/No — Detail if any]

### 4.3 Unique Value & Novel Insights:
- [List any genuine biological or methodological insights contributed by this review]

### 4.4 Review Limitations:
- [Disclose any blind spots, reasoning shortcuts, or platform constraints]
