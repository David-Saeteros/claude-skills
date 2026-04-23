---
name: review-research
description: >
  Pre-submission academic paper review and full PhD thesis manuscript review for
  personality psychology and NLP research. Use this skill whenever the user wants
  to review a paper or thesis, run a mock referee report, simulate a defence
  committee, or get structured feedback on a LaTeX manuscript. Triggers on phrases
  like "review my paper", "review my thesis", "pre-submission check", "mock referee
  report", "defence feedback", or any mention of reviewing a .tex manuscript. Domain:
  personality psychology, NLP, transformer models, XAI (Integrated Gradients), OCEAN/
  Big Five. Supported journals: PLOS_ONE, EJPA, PAID, JAP, Psych_Methods.
---

# review-research

A rigorous pre-submission and thesis review skill for personality psychology and NLP
research. Seven specialised agents run in parallel and their findings are consolidated
into a single structured report in English.

---

## Invocation

```
/review-research                             # auto-detect .tex, generic standards
/review-research thesis                      # full manuscript, defence committee persona
/review-research PAID                        # paper review, PAID journal persona
/review-research JAP path/to/main.tex        # explicit path + journal persona
/review-research Psych_Methods               # paper review, Psychological Methods persona
```

**Recognised journal handles** (case-insensitive):

| Handle | Journal |
|---|---|
| `PLOS_ONE` | PLOS ONE |
| `EJPA` | European Journal of Psychological Assessment |
| `PAID` | Personality and Individual Differences |
| `JAP` | Journal of Applied Psychology |
| `Psych_Methods` | Psychological Methods |

If no journal is specified and mode is not `thesis`, the review applies high general
standards without a specific journal persona.

---

## Phase 1 — Setup

1. **Detect mode**: if the first argument is `thesis`, set mode to THESIS. Otherwise set
   mode to PAPER.
2. **Detect file**: if a `.tex` path is provided, use it. Otherwise search the current
   directory for the main `.tex` file (the one containing `\documentclass` or
   `\begin{document}`). If multiple candidates are found, ask the user to confirm.
3. **Read the manuscript**: load the full `.tex` source. Also read any referenced `.bib`
   file(s) for citation checks.
4. **Adopt persona**:
   - PAPER mode + journal specified → adopt that journal's referee persona (see Personas below)
   - PAPER mode + no journal → rigorous generalist reviewer
   - THESIS mode → three-member defence committee (see Thesis Mode below)

---

## Phase 2 — Parallel Agent Reviews

Spawn all 7 agents simultaneously. Each agent receives the full manuscript and
produces a structured section for the final report.

### Agent 1 — Language, Style & APA Formatting
- Academic register and clarity of writing
- Grammar, syntax, and readability
- APA 7th edition compliance: in-text citations, reference list format, headings,
  tables, figures, statistics reporting (italics, spaces, decimal places)
- Consistency of terminology (e.g., "personality traits" vs "personality dimensions"
  used interchangeably without justification)
- Overuse of hedging or overclaiming language

### Agent 2 — Internal Consistency & Cross-References
- All in-text citations present in the reference list and vice versa
- Figure and table numbers match their in-text callouts
- Hypotheses stated in the introduction are addressed in results and discussion
- Abbreviations defined on first use and used consistently
- Section cross-references (e.g., "as shown in Section 3") are accurate
- Numbering of equations, tables, and figures is sequential and correct

### Agent 3 — Unsupported Claims & Construct Validity
- Claims not backed by citations or data
- Overgeneralisation from sample to population
- Construct validity of personality measures: is the operationalisation of OCEAN/Big
  Five traits theoretically justified?
- Are the essay prompts (e.g., Pennebaker paradigm) appropriate for the constructs
  being measured?
- Discriminant validity: are distinctions between traits (e.g., Openness vs
  Intellect) adequately addressed?

### Agent 4 — NLP Methodology
- Appropriateness of transformer model choice (BERT, RoBERTa, domain-specific variants)
- Tokenisation strategy and handling of out-of-vocabulary tokens
- Train/validation/test split rationale and potential data leakage
- Baseline comparisons: are classical NLP methods (TF-IDF, LIWC) included where relevant?
- Evaluation metrics: are chosen metrics (r, MAE, F1, etc.) appropriate for the task?
- Reproducibility: are hyperparameters, seeds, and model versions reported?
- Pre-training vs fine-tuning strategy: is the choice justified?

### Agent 5 — XAI Validity (Integrated Gradients)
- Faithfulness of attribution method: does IG actually reflect model behaviour or is
  it a post-hoc rationalisation?
- Baseline selection for IG: is the choice of reference input (e.g., zero embedding,
  mask token) justified and its effect on attributions discussed?
- Sensitivity analysis: are attributions stable across runs and input perturbations?
- Interpretability claims: does the paper overinterpret token-level attributions as
  psychological meaning?
- Sanity checks: are there ablation studies or correlation analyses validating that
  high-attribution tokens correspond to meaningful linguistic features?
- Comparison to alternative XAI methods (SHAP, LIME, attention) where relevant

### Agent 6 — Psychometrics
- Reliability of personality scores used as labels (internal consistency, test-retest
  where available)
- Convergent and discriminant validity evidence for the measures
- Effect size reporting: are r, d, or η² values provided alongside significance tests?
- Sample size and power: is the study adequately powered for the claimed effects?
- Treatment of the Big Five as continuous vs categorical: is this consistent and
  justified?
- Correlation between traits (multicollinearity) acknowledged where relevant
- Appropriate use of correction for multiple comparisons

### Agent 7 — Contribution & Novelty (Persona-Driven)

**PAPER mode — Journal Referee Persona:**
Adopt the editorial stance of the target journal (see Personas below). Evaluate:
- Novelty relative to existing literature
- Fit with the journal's scope and readership
- Whether the contribution claim is proportionate to the evidence
- Likely accept/major revision/reject recommendation with justification

**THESIS mode — Defence Committee Persona:**
Three voices review holistically:
- *Methodologist*: rigour of research design, statistical and computational choices,
  replicability
- *Domain Expert*: theoretical contribution to personality psychology and NLP,
  positioning within the literature
- *External Examiner*: overall coherence of the manuscript, chapter-to-chapter
  narrative flow, strength of conclusions relative to findings, viva readiness

Thesis-specific checks:
- Introduction → Literature Review → Methods → Results → Discussion → Conclusion
  narrative arc is coherent
- Research questions stated in Chapter 1 are explicitly answered in Discussion/Conclusion
- Limitations are acknowledged honestly and not relegated to a single perfunctory paragraph
- Contributions are clearly enumerated and proportionate

---

## Phase 3 — Consolidation

Merge all 7 agent outputs into a single structured report with the following sections:

```
# RESEARCH REVIEW — [YYYY-MM-DD]
## Manuscript: [filename or title]
## Mode: [PAPER — Journal / THESIS — Defence Committee]

## Executive Summary
[3–5 sentence overall assessment and recommendation]

## Agent 1: Language, Style & APA
## Agent 2: Internal Consistency & Cross-References
## Agent 3: Unsupported Claims & Construct Validity
## Agent 4: NLP Methodology
## Agent 5: XAI Validity
## Agent 6: Psychometrics
## Agent 7: Contribution & Novelty [or Defence Committee Assessment]

## Priority Action List
[Numbered list of the most critical issues to address, ranked by severity]
```

Within each agent section, use three severity levels:
- 🔴 **Critical** — must be addressed before submission/defence
- 🟡 **Major** — significant weakness, strongly recommended to fix
- 🟢 **Minor** — improvement suggested but not blocking

Save the report as `RESEARCH_REVIEW_[YYYY-MM-DD].md` in the current directory.

---

## Journal Personas

### PLOS ONE
Focuses on scientific and methodological rigour rather than novelty or impact.
Referee concerns: transparency, reproducibility, appropriate statistical reporting,
open data/code availability.

### EJPA
Focused on psychological assessment methodology. High bar for psychometric quality,
construct validity, and measurement precision. Sceptical of NLP approaches unless
thoroughly validated against established assessment instruments.

### PAID
Broad personality research scope. Values individual differences framing, adequate
sample sizes, and replication. Sceptical of overclaiming from text-based inference;
expects discussion of ecological validity.

### JAP
Applied focus. Expects practical implications and relevance to organisational or
real-world settings. High bar for incremental validity over existing measures.

### Psych_Methods
Methodologically rigorous. Deep scrutiny of statistical choices, model assumptions,
effect size estimation, and computational reproducibility. Expects formal treatment
of all methodological decisions.

---

## Notes for Claude Code

- Always read the full `.tex` source before spawning agents — do not summarise or
  truncate the input to agents.
- If the manuscript references external files (figures, tables, supplementary `.tex`
  files via `\input` or `\include`), attempt to read those too.
- The `.bib` file is essential for Agent 2. If not found in the current directory,
  search one level up.
- All output is in **English** regardless of the manuscript language.
- Do not include page numbers or LaTeX line references in the report — use section
  headings and quote short phrases for localisation instead.
