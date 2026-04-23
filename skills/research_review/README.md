# review-research

A Claude Code skill for rigorous pre-submission paper review and full PhD thesis
manuscript review, tailored for personality psychology and NLP research.

---

## What it does

Spawns 7 specialised review agents in parallel, each focusing on a different
dimension of quality. Their findings are consolidated into a single structured
Markdown report saved to your project directory.

---

## Agents

| # | Agent | Focus |
|---|---|---|
| 1 | Language, Style & APA | Grammar, academic register, APA 7th compliance |
| 2 | Internal Consistency | Cross-references, citation integrity, figure/table alignment |
| 3 | Claims & Construct Validity | Unsupported claims, OCEAN/Big Five operationalisation |
| 4 | NLP Methodology | BERT/RoBERTa usage, splits, baselines, reproducibility |
| 5 | XAI Validity | Integrated Gradients attribution quality and interpretability claims |
| 6 | Psychometrics | Reliability, validity, effect sizes, power |
| 7 | Contribution & Novelty | Journal referee persona OR defence committee (thesis mode) |

---

## Invocation

From within your paper or thesis project directory:

```
/review-research                             # auto-detect .tex, generic standards
/review-research thesis                      # full manuscript, defence committee
/review-research PAID                        # PAID journal referee persona
/review-research JAP path/to/main.tex        # explicit path + JAP persona
/review-research Psych_Methods               # Psychological Methods persona
```

**Recognised journal handles:** `PLOS_ONE`, `EJPA`, `PAID`, `JAP`, `Psych_Methods`

---

## Output

A consolidated report saved as:

```
RESEARCH_REVIEW_YYYY-MM-DD.md
```

Issues are flagged at three severity levels:
- 🔴 Critical — must fix before submission/defence
- 🟡 Major — strongly recommended to fix
- 🟢 Minor — improvement suggested

---

## Requirements

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code)
- The Agent tool must be available (for parallel subagents)
- Manuscript in LaTeX format (`.tex`)

---

## Installation

Copy `review-research.md` to your Claude Code commands directory:

```bash
# Global installation
cp review-research.md ~/.claude/commands/review-research.md

# Project-level installation
cp review-research.md .claude/commands/review-research.md
```

---

## Part of the `claude-skills` family

This skill is domain-specific to personality psychology + NLP research and
pairs well with the `academic-writing`, `paper-companion`, and `ra-fieldwork` skills.
