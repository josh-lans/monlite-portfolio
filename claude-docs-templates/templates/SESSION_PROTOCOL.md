# Session Protocol

> **Purpose:** Rules for how AI sessions should operate, including subagent delegation and task workflows.

---

## Session Start Checklist

1. Read `MEMORY.md` (current state, versions, anti-shortcuts)
2. Read `STANDARDS.md` (mandatory patterns)
3. Check git: `git branch --show-current && git log --oneline -5`
4. Check for uncommitted changes: `git status`
5. Ask what we're working on today

## Session End Checklist

1. Commit all changes with descriptive messages
2. Update `MEMORY.md` with current state
3. Add entry to `SESSION_LOG.md`
4. Add any new lessons to `LESSONS.md`
5. Note anything left incomplete

## Subagent Delegation

When using parallel agents (e.g., Claude Code's Agent tool), follow these rules:

- **Give full context** — Each agent starts fresh. Include file paths, what to look for, expected patterns.
- **Specify scope** — "Audit files A, B, C for pattern X" not "check if everything is okay"
- **Research vs. implementation** — Clearly state if the agent should read-only or make changes
- **Parallel when independent** — Launch multiple agents simultaneously for independent tasks
- **Sequential when dependent** — Wait for research results before implementation

## Task Complexity Guidelines

| Complexity | Approach |
|-----------|----------|
| Single file fix | Just do it |
| Multi-file feature | Use todo list to track progress |
| Architecture change | Enter plan mode, design first, get approval |
| Cross-cutting audit | Launch parallel research agents |

## Commit Standards

- Descriptive messages explaining WHY, not just WHAT
- Reference related issues/sessions if applicable
- One logical change per commit (not "fix everything")
- Run tests before committing
