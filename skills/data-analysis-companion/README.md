# data-analysis-companion

A structured data analysis skill for Claude. Turns Claude into a thinking partner for exploring, cleaning, and analysing data in **R** or **Python** — not just a code generator.

---

## What it does

The skill follows a two-act structure:

**Act 1 — Blind Exploration**
Claude interviews you about your data, runs a structural scan (shape, variable types, missingness, distributions), writes a plain-language data portrait, and surfaces a menu of analytical paths worth considering — all before any research question is needed.

**Act 2 — Targeted Analysis**
Once you have a research question, Claude matches it to the right analytical family, checks assumptions first, and works through the analysis one step at a time: propose code → you run it → paste output → Claude interprets → iterate.

---

## Analytical families covered

| Family | Coverage |
|---|---|
| Survey & observational data | Likert handling, group comparisons, regression, experimental designs |
| Psychometrics | Reverse scoring, reliability (α, ω), EFA, CFA, scale scores, validity |
| Econometrics | OLS with robust SEs, panel FE/RE, DiD (incl. staggered), IV, RD |
| Longitudinal | Mixed models, LGM, repeated measures, multilevel nesting |
| NLP — general | Preprocessing, TF-IDF, sentiment, topic modelling (LDA, BERTopic), embeddings |
| NLP — transformers & XAI | BERT-family classification, Integrated Gradients, attribution aggregation, personality inference |

---

## Adaptive depth

The skill is honest about what it knows well vs. where it's at the edge. For core methods it goes deep and specific. For specialised edge cases it scaffolds an approach and points toward the right resources.

---

## Trigger phrases

The skill activates on phrases like:

- "help me analyse this data"
- "explore this dataset"
- "clean my data"
- "run some descriptives"
- "what can I do with this"
- "help me pick a model"
- "interpret these results"

Also triggers on analytical context keywords: survey data, Likert scales, factor analysis, regression, psychometrics, econometrics, longitudinal data, panel data, NLP, topic modelling, transformer models, Integrated Gradients.

---

## File structure

```
data-analysis-companion/
├── SKILL.md                          # Core protocol and routing logic
└── references/
    ├── survey-social-science.md      # Surveys, group comparisons, regression
    ├── psychometrics.md              # Scale validation, EFA/CFA, reliability
    ├── econometrics.md               # Causal inference, panel data, DiD, IV, RD
    ├── longitudinal.md               # Mixed models, growth curves, multilevel
    └── nlp-text.md                   # General NLP + transformer/XAI deep layer
```

---

## Installation

Unzip and place the `data-analysis-companion/` folder in your skills directory (e.g. `skills/user/`). Claude will pick it up automatically on the next session.
