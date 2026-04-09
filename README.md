# claude-skills

A collection of skills for [Claude](https://claude.ai) — structured instruction sets that extend Claude's behaviour for specialised workflows.

Built and maintained by **David Saeteros** ([@saeteros_david](https://twitter.com/saeteros_david)).

---

## What is a skill?

A skill is a markdown file (`SKILL.md`) that gives Claude detailed, domain-specific instructions for a particular type of task. When installed in a Claude Project, Claude reads the skill and follows its workflow automatically — no prompting required.

Skills can include:
- A main `SKILL.md` with mode detection, step-by-step workflows, and decision logic
- `references/` files loaded on demand for extended guidance (section templates, style guides, etc.)

---

## Skills

| Skill | Description | Status |
|-------|-------------|--------|
| [academic-writing](skills/academic-writing/) | Draft, edit, structure, cite, respond to reviewers, and review thesis manuscripts against supervisor feedback | ✅ Stable |

---

## How to install a skill

1. Open [Claude.ai](https://claude.ai) and go to a **Project**
2. In the Project settings, find **Custom Instructions** or the skill upload area
3. Upload the `SKILL.md` file (and optionally the `references/` folder) from the skill you want
4. Claude will use the skill automatically for relevant requests in that Project

> Skills work best inside a dedicated Claude Project scoped to that domain.

---

## Contributing

Contributions welcome — new skills, improvements to existing ones, or bug reports via Issues.

If you build a skill using this repo as a base, attribution appreciated (see licence below).

---

## Licence

This work is licensed under [Creative Commons Attribution 4.0 International (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/).

You are free to use, adapt, and redistribute these skills for any purpose, including commercially, as long as you give appropriate credit to the original author.

**Attribution format:**  
> Skills by David Saeteros — https://github.com/David-Saeteros/claude-skills
