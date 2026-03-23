# AI-Assisted Development Methodology

## Overview

This document describes the structured methodology I developed for building enterprise software with AI coding assistants. This approach enabled a single developer to build a production-grade monitoring platform (19 SAP surveillance categories, 7 database engines, distributed collectors, AI diagnostics) in 6 weeks.

## The Problem

AI coding assistants are powerful but have critical limitations:
1. **Context amnesia** — Each session starts fresh, losing architectural decisions
2. **Shortcut tendency** — Without constraints, AI takes the simplest path, not the correct one
3. **Scale blindness** — AI doesn't inherently understand that code running against 3,000 SAP systems needs different patterns than a prototype

## The Solution: Structured Session Memory

### File Structure

```
.claude-docs/
├── MEMORY.md           # Essential context — read first every session
├── STANDARDS.md        # 50+ mandatory patterns with § references
├── DEPENDENCIES.md     # Change impact checklists
├── SESSION_LOG.md      # Detailed session history
├── SESSION_PROTOCOL.md # Subagent delegation rules
├── LESSONS.md          # Learned patterns (never repeat mistakes)
└── [domain-specific].md # Architecture docs per module
```

### MEMORY.md — The "Brain"

Contains critical context that must survive across sessions:
- Current project state and recent changes
- Environment details (ports, paths, database config)
- Anti-shortcut protocols ("NEVER implement a simpler version of what the architecture doc specifies")
- Current version numbers for all components
- Known gotchas that caused rework

### STANDARDS.md — The "Rulebook"

50+ numbered development patterns referenced by section (§):
- **§50**: End-to-end wiring checklist for new features
- **§38**: Every metric needs a remediation registry entry
- **§14**: Never query in loops — batch fetch with ID maps
- **§7b**: New metric wiring: registry → collector → storage → alarm → remediation → test → webhook
- **§11**: Collector version bump on ANY collector file change

### DEPENDENCIES.md — The "Impact Matrix"

When changing X, you MUST also change Y:
- Adding a new surveillance category requires updates in 6+ files
- Adding a new database engine requires driver + metric_registry + help docs + seed thresholds
- Changing an alarm metric name requires remediation_registry + webhook + Luna context

### LESSONS.md — The "Scar Tissue"

Real mistakes that caused rework, documented to prevent recurrence:
- "FIELD_NOT_VALID on RFC_READ_TABLE is usually DATA_BUFFER overflow, not missing fields"
- "NPL is NOT a stripped-down SAP system — always verify via SE16 before assuming fields don't exist"
- "Test checks don't persist to DB — only regular collection cycles store data"

## Key Principles

### 1. Read Before Write
Every session starts by reading MEMORY.md, STANDARDS.md, and checking git state. No code is written before understanding current context.

### 2. Architecture-First
Complex features use Plan Mode — explore the codebase, design the approach, get approval, then implement. This prevents the most expensive mistake: building the wrong thing correctly.

### 3. Parallel Audit Agents
For cross-cutting concerns (interval consistency, help text accuracy, remediation coverage), launch multiple specialist agents in parallel to audit different parts of the codebase simultaneously.

### 4. Incremental Validation
Every change is tested immediately — build, deploy, verify in the running system. No "implement everything then test" batches.

### 5. Todo Lists for Complex Tasks
Multi-step tasks use explicit todo lists tracked in the conversation. Each item is checked off as completed. This prevents scope drift and ensures nothing is forgotten.

## Results

- **160+ development sessions** with consistent quality
- **524 passing tests** at time of ABAP RFC feature merge
- **Zero production outages** from code deployed to monitored SAP systems
- **19 SAP surveillance categories** validated end-to-end against live SAP NPL
- **Full audit trail** — every session's changes documented with rationale

## Applicability

This methodology works for any domain where:
- The developer has deep domain expertise but may lack traditional software engineering background
- The codebase is too large for the AI to hold in context at once
- Quality requirements are high (enterprise, production systems)
- Development spans weeks or months (not a one-shot script)

The key insight: **AI doesn't replace expertise — it amplifies it.** The domain knowledge (knowing which SAP RFC function modules to call, what fields TPALOG has, why WP percentage rules need state filtering) comes from the human. The AI accelerates translating that knowledge into working software.
