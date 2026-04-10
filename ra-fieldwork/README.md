# ra-fieldwork

Part of the **[research-assistant](../research-assistant)** skill family.

Handles the manual, repetitive, copy-paste-transfer labour of research work — the unglamorous fieldwork that would otherwise be done by hand.

---

## What it does

| Task | Example |
|---|---|
| **PDF → Excel extraction** | Pull structured fields from one or many papers into a new or existing spreadsheet |
| **Excel → Excel transfer** | Copy data from a supplemental material file into your working database, respecting its existing schema |
| **Codebook / data dictionary** | Generate variable labels, types, and value descriptions from a dataset |
| **Literature matrix** | Build an extraction table from a set of papers |
| **Cross-checking / auditing** | Verify citations match the reference list, variables match the codebook, entries match the protocol |
| **Simplification** | Convert academic or technical text into plain structured output |

---

## When to use it

Use this skill when you need Claude to move, extract, or organise information between documents — tasks you would otherwise do manually by copying and pasting. It works on any domain and any document type.

Trigger phrases:
- "extract this from the PDF and put it in my Excel"
- "add this to my existing spreadsheet"
- "build a codebook from this dataset"
- "check if these citations match the reference list"
- "make a table from these papers"
- "pull this from the supplemental material"

---

## How to install

1. Go to your Claude Project on [claude.ai](https://claude.ai)
2. Open Project settings → Custom Instructions
3. Upload `SKILL.md` from this folder
4. Claude will apply the skill automatically for relevant requests

> For best results, also install the full **research-assistant** parent skill so Claude can route between all RA sub-skills automatically.

---

## Dependencies

This skill delegates file operations to two public Claude skills:
- [`pdf-reading`](https://github.com/anthropics/claude-skills) — for reading and extracting from PDFs
- [`xlsx`](https://github.com/anthropics/claude-skills) — for reading and writing Excel files

These are built-in public skills and do not need to be installed separately.

---

## Part of the research-assistant family

| Skill | Role | Status |
|---|---|---|
| `research-assistant` | Parent router | 🔲 Coming soon |
| `ra-fieldwork` | Copy-extract-transfer between files | ✅ This skill |
| `ra-writing` | Draft and edit academic sections | 🔲 Coming soon |
| `ra-synthesis` | Read and compare papers | 🔲 Coming soon |
| `ra-screening` | Search strings and abstract screening | 🔲 Coming soon |
| `ra-qc` | Review drafts against rubrics | 🔲 Coming soon |
