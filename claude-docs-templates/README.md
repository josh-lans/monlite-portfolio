# `.claude-docs` Templates — Structured AI Development

> **Drop-in templates for maintaining project context across AI coding sessions.**

These templates implement the methodology described in [AI_DEVELOPMENT_METHODOLOGY.md](../methodology/AI_DEVELOPMENT_METHODOLOGY.md). They're designed for Claude Code but work with any AI coding assistant that reads project files.

## Quick Start

```bash
mkdir -p .claude-docs
cp templates/MEMORY.md .claude-docs/
cp templates/STANDARDS.md .claude-docs/
cp templates/DEPENDENCIES.md .claude-docs/
cp templates/SESSION_LOG.md .claude-docs/
cp templates/LESSONS.md .claude-docs/
```

Then add to your `CLAUDE.md` (or equivalent project root config):

```markdown
## FIRST ACTION: Read Documentation
Before writing ANY code, read these files in order:
1. `.claude-docs/MEMORY.md`
2. `.claude-docs/STANDARDS.md`
3. `.claude-docs/DEPENDENCIES.md`
```

## Templates

| File | Purpose | Update Frequency |
|------|---------|-----------------|
| **[MEMORY.md](templates/MEMORY.md)** | Essential working context — the AI reads this first every session | Every session |
| **[STANDARDS.md](templates/STANDARDS.md)** | Numbered development patterns the AI must follow | When patterns are established |
| **[DEPENDENCIES.md](templates/DEPENDENCIES.md)** | Change impact checklists — "if you change X, also change Y" | When new features are wired |
| **[SESSION_LOG.md](templates/SESSION_LOG.md)** | Session history for context continuity | End of every session |
| **[LESSONS.md](templates/LESSONS.md)** | Mistakes that caused rework — never repeat these | When mistakes happen |
| **[SESSION_PROTOCOL.md](templates/SESSION_PROTOCOL.md)** | Rules for subagent delegation and task workflows | Once, then rarely |
| **[CLAUDE.md](templates/CLAUDE.md)** | Project root file — auto-read by Claude Code | Once, then update rules as needed |

## How It Works

### The Problem
AI coding assistants lose context between sessions. Without structure, they:
- Forget architectural decisions made weeks ago
- Take shortcuts that violate established patterns
- Miss wiring steps when adding features (metric added but alarm not connected)
- Repeat mistakes that were already solved

### The Solution
These files create a **persistent knowledge layer** that survives across sessions:

1. **MEMORY.md** is the brain — current state, environment, version numbers, anti-shortcut rules
2. **STANDARDS.md** is the rulebook — numbered patterns referenced by `§` (e.g., "per §7b, new metrics require 8 wiring steps")
3. **DEPENDENCIES.md** is the impact matrix — prevents the "I changed X but forgot Y" class of bugs
4. **LESSONS.md** is scar tissue — real mistakes documented so they never happen again
5. **SESSION_LOG.md** is the timeline — what changed, when, and why

### Key Principles

- **Read before write** — Every session starts by reading MEMORY + STANDARDS
- **Number everything** — Standards use `§` references so the AI can cite specific rules
- **Document mistakes immediately** — LESSONS.md entries prevent the most expensive bugs (rework)
- **Update MEMORY every session** — Stale context = stale decisions
- **Keep SESSION_LOG detailed** — Future sessions need to know what happened and why

## Adapting for Your Project

The templates are intentionally generic. Customize them:

1. **MEMORY.md** — Replace placeholder sections with your actual project context
2. **STANDARDS.md** — Start with 5-10 patterns, add more as they're established
3. **DEPENDENCIES.md** — Map your own change impact chains
4. **LESSONS.md** — Start empty, populate as you learn
5. **CLAUDE.md** — Adapt the critical rules section to your project's pitfalls

## Works With

- **Claude Code** (Anthropic) — reads `CLAUDE.md` automatically, `.claude-docs/` by convention
- **Cursor** — add files to `.cursorrules` or reference in project config
- **GitHub Copilot** — reference files in chat context
- **Any AI assistant** — just paste the file contents at the start of each session
