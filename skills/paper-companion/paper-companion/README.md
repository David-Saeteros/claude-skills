# paper-companion

A Claude skill that guides you through any academic paper using a structured four-stage reading protocol — warm, rigorous, and built around pattern recognition rather than passive summarising.

---

## What it does

Every paper gets walked through four stages, one at a time:

| Stage | Goal |
|-------|------|
| **1 · Structural Scan** | Build the mental map — headings, abstract, figures, study type |
| **2 · Core Findings** | Extract what they found and make every statistic mean something |
| **3 · Critical Evaluation** | Assess strengths, weaknesses, and overreach — independently, not just echoing the authors |
| **4 · Reading Note** | Consolidate into a permanent note card ready to paste into Zotero |

### Design principles

- **Never dumps everything at once** — one stage per response, conversational pace
- **Always translates statistics** — every number gets a plain-language explanation
- **Supervised highlighting** — at each stage, tells you exactly what to mark in the PDF and why, building the instinct over time
- **Adapts to you** — reads from Claude's memory if available; otherwise asks two quick questions once and proceeds
- **Flags connections proactively** — links the paper to your own work and prior papers you've mentioned

---

## Files

```
paper-companion/
└── SKILL.md    # Core skill — install this in your Claude Project
```

---

## Installation

1. Go to your Claude Project on [claude.ai](https://claude.ai)
2. Upload `SKILL.md` to the Project's custom instructions
3. Upload a paper PDF — the skill starts automatically

---

## Example prompts

```
[Upload PDF] — skill starts Stage 1 automatically

Let's read this paper together.

Walk me through this study.

I'm at Stage 3 — what should I be looking for?
```

---

## Note card output

Stage 4 produces a plain-text note card designed to paste cleanly into Zotero or any notes field:

```
PAPER NOTE

Reference: Author(s), Year, Full title, Journal/venue

Core argument: ...

Key findings:
- ...
- ...
- ...

Method: ...

Strength: ...

Weakness: ...

My verdict: ...

Possible connections: ...
```

---

## Licence

[CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) — David Saeteros · https://github.com/David-Saeteros/claude-skills
