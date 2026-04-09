# academic-writing

A Claude skill for expert academic writing assistance across any genre — journal articles, theses, grant proposals, systematic reviews, and book chapters.

---

## What it does

The skill detects which mode you need and applies the right workflow automatically:

| Mode | What it handles |
|------|----------------|
| **Draft** | Generates section text from scratch with `[CITATION NEEDED]` markers and a self-audit |
| **Edit** | Flags precision, rigour, tone, and structural issues — then rewrites only what you confirm |
| **Structure** | Maps your argument, diagnoses logic gaps, proposes a revised skeleton |
| **Citation** | Formats and audits references in APA 7th (or any style you specify) |
| **Reviewer response** | Triages reviewer comments, calibrates tone, drafts a point-by-point rebuttal letter |
| **Thesis review** | Reconciles supervisor feedback with your manuscript — outputs a prioritised change table with exact locations and instructions |

### Design principles

- **Preserve voice, enforce rigour** — edits keep your tone intact but push back hard on imprecision, causal overclaiming, and missing evidence
- **Flag first, change second** — surfaces issues and asks before rewriting anything substantive
- **APA 7th by default** — switches to any other guide on request
- **Always catches** causal language on correlational data, missing effect sizes, non-statistical use of "significant", and undefined abbreviations — regardless of what you asked for

---

## Files

```
academic-writing/
├── SKILL.md                          # Core skill — install this in your Claude Project
└── references/
    ├── section-guides.md             # Drafting guidance per section (intro, methods, results, etc.)
    └── apa7-extended.md              # Full APA 7th reference type catalogue
```

---

## Installation

1. Go to your Claude Project on [claude.ai](https://claude.ai)
2. Upload `SKILL.md` to the Project's custom instructions
3. Optionally add the `references/` files — Claude will read them when needed

---

## Example prompts

```
Edit this discussion paragraph for clarity and rigour: [paste text]

Draft a methods section for a cross-sectional survey study on personality and wellbeing,
targeting PLOS ONE, ~400 words.

Fix this reference list for APA 7th: [paste list]

I have supervisor feedback on my thesis [upload PDF] and my manuscript [upload .tex].
Give me a prioritised change table.

Help me respond to this reviewer comment: [paste comment]
```

---

## Licence

[CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) — David Saeteros · https://github.com/David-Saeteros/claude-skills
