# Project AI Session Memory

> **Purpose:** Essential working context. Read this first in every new session.
>
> **Last Updated:** YYYY-MM-DD (Session N: brief description of what changed)

---

## Project Overview

<!-- One paragraph: what is this project, what does it do, who is it for -->

## Current State

<!-- What's the current development focus? What branch are we on? What's in progress? -->

- **Branch:** `main` / `feature/xxx`
- **Current focus:** [describe]
- **Blocked on:** [if anything]

## Environment

<!-- Paths, ports, database, runtime versions — everything the AI needs to find things -->

| Item | Value |
|------|-------|
| **Working directory** | `/path/to/project` |
| **Backend port** | 8000 |
| **Frontend port** | 3000 |
| **Database** | PostgreSQL 16 on localhost:5432 |
| **Python** | 3.12 |
| **Node** | 20.x |

## Version Numbers

<!-- Bump these when versions change — the AI needs to know current versions -->

| Component | Current Version |
|-----------|----------------|
| Backend | v1.0.0 |
| Frontend | v1.0.0 |
| Collector/Agent | v1.0.0 |

## Anti-Shortcut Protocol

<!-- Rules that prevent the AI from taking shortcuts. Add to this list as you discover patterns. -->

- **NEVER** implement a "simpler version" of what the architecture document specifies
- **NEVER** skip reading the architecture doc because "I remember the design"
- **NEVER** query in loops — batch fetch with ID maps
- **ALWAYS** bump version numbers when changing component files
- **ALWAYS** run tests before committing

## Connector/Module Updates

<!-- Patterns specific to your project's update workflows -->

- **ALWAYS bump `COMPONENT_VERSION`** when changing component files (current: **v1.0.0**)
- **ALWAYS** say: "Push the update from the UI" (never tell the user to SSH in and edit files manually)

## Recent Context

<!-- Last 2-3 sessions of context for continuity -->

### Last Session
- [What was done]
- [What was left incomplete]
- [Any decisions that were made]
