# Project Development — Mandatory Context

> **This file is read automatically at the start of every Claude Code session.**
> **Adapt this template to your project's specific needs.**

## FIRST ACTION: Read Documentation

Before writing ANY code, you MUST read these files in order:

1. `.claude-docs/MEMORY.md` — project context, environment, gotchas
2. `.claude-docs/STANDARDS.md` — mandatory development patterns (§ references)
3. `.claude-docs/DEPENDENCIES.md` — change impact checklists

Check git branch and recent commits: `git branch --show-current && git log --oneline -5`

## Working Style

<!-- Describe how you like to work with the AI. Examples: -->

- Be direct and efficient — no unnecessary preamble
- When something breaks, diagnose fast and fix it
- Don't over-apologize for mistakes — just fix them
- Use todo lists for multi-step tasks
- Plan complex features before implementing

## CRITICAL RULES

<!-- These prevent the most common/expensive mistakes. Add your own. -->

### General
- **Version bump** on ANY change to [component] files. Current: **v1.0.0**
- **Run tests before committing**: `[your test command]`
- **Every new [metric/feature] needs full wiring**: [list your wiring steps]
- **Never query in loops** — batch fetch + ID maps
- **Feature branches** for non-trivial changes

### Domain-Specific
<!-- Add rules specific to your project's domain. Examples: -->
- [Rule about your data model]
- [Rule about your API contract]
- [Rule about your deployment process]

## WHAT NOT TO DO

<!-- Past mistakes that caused rework — the AI reads these as hard constraints -->

- Do NOT implement a "simpler version" of what the architecture doc specifies
- Do NOT skip reading the architecture doc because "I remember the design"
- Do NOT merge to main without review
- Do NOT [your project-specific anti-patterns]

## PROJECT IDENTITY

<!-- Help the AI understand the product vision and quality bar -->

[Your project] is [brief description]. The target audience is [who].
Quality bar: [describe]. No shortcuts, no amateur implementations.
