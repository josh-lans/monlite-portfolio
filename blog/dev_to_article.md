---
title: How I Built Enterprise Monitoring Software in 4 Months Using Structured AI Development
published: true
tags: ai, programming, productivity, saas
---

## The Problem

I'm a SAP Basis administrator with 12 years of experience managing enterprise infrastructure. I know exactly what monitoring tools should do — I've used (and been frustrated by) every commercial option: SolarWinds, Avantra, Solution Manager, RedPeaks. They're either too expensive, too rigid, or don't understand SAP deeply enough.

So I decided to build my own. The catch: I'm not a traditional software developer. I understand systems architecture and can read code, but I've never built a full-stack web application from scratch.

## The Approach: AI as Force Multiplier

I used **Claude Code** (Anthropic's CLI agent) as my primary development partner, with OpenAI Codex, Gemini, and GitHub Copilot for independent verification. But the key insight wasn't the AI — it was the **methodology** I developed to make AI-assisted development work at enterprise scale.

### The `.claude-docs` System

After the first few sessions, I hit a wall. The AI would forget architectural decisions between sessions. It would take shortcuts that broke patterns established weeks ago. It would implement a "simpler version" of what we'd carefully designed.

I solved this with a structured documentation system:

```
.claude-docs/
├── MEMORY.md           # Essential context — read first every session
├── STANDARDS.md        # 50+ mandatory patterns with § references
├── DEPENDENCIES.md     # Change impact checklists
├── SESSION_LOG.md      # Session history for continuity
├── LESSONS.md          # Never repeat these mistakes
└── [domain-specific].md
```

**MEMORY.md** contains everything the AI needs to know before writing a single line of code: current project state, environment details, version numbers, and explicit anti-shortcut rules.

**STANDARDS.md** has 50+ numbered development patterns. When the AI needs to add a new metric, it references §7b which lists all 8 files that need updating. When it needs to add a threshold, §7 lists the 6-step checklist. No shortcuts, no missing steps.

**DEPENDENCIES.md** is the impact matrix. "If you change X, you MUST also change Y." Adding a new SAP surveillance category requires updates in 6+ files across backend, frontend, collector, and documentation. Miss one and something silently breaks.

**LESSONS.md** is scar tissue from real mistakes:
- *"RFC_READ_TABLE FIELD_NOT_VALID is usually DATA_BUFFER overflow, not missing fields — verify via SE16 before assuming"*
- *"Test checks don't persist to DB — only regular collection cycles store data"*
- *"Unified connectors need to be included in BOTH sapcontrol AND abap_rfc entity queries"*

### Multi-AI Verification

For security-critical features, I ran independent reviews across multiple AI platforms. Claude would implement a feature, then I'd have Codex audit it for OWASP vulnerabilities, Gemini review the architecture, and Copilot check for common patterns. Each AI catches different classes of issues.

This resulted in an 8-phase security hardening that covers OWASP Top 10, with features like:
- HttpOnly cookie sessions (no localStorage tokens)
- CSRF double-submit protection
- SSRF fail-closed on DNS errors
- HMAC-signed audit logs
- Rate limiting that fails closed in production

## What I Built

**MonLite** — a full enterprise monitoring platform:

- **SAP Monitoring**: 19 surveillance categories via ABAP RFC + SAPControl SOAP, with a rules engine (not just thresholds) for per-entity evaluation
- **Database Monitoring**: 7 engines (MSSQL, Oracle, HANA, DB2, Sybase, MaxDB, PostgreSQL) with per-engine specialized drivers
- **Host Monitoring**: Linux (SSH) and Windows (WinRM) with predictive storage projections
- **AI Diagnostics**: An air-gapped LLM assistant that queries live infrastructure data
- **Enterprise Security**: SSO (Okta/Entra ID), RBAC, encrypted credentials, signed audit logs

The frontend is a React 19 app with a glassmorphism design language. The backend is FastAPI + PostgreSQL. Distributed Python collectors run at the edge behind firewalls.

It's designed for 3,000+ monitored endpoints with 200+ concurrent users.

## Key Lessons

### 1. Domain expertise is the bottleneck, not coding ability

The AI can write Python, React, and SQL faster than I ever could. But it can't tell you that SAP's `TH_WPINFO` RFC captures its own work process in the results, that `TPALOG` has a combined timestamp field instead of separate date/time, or that `INST_EXECUTE_REPORT` is the only reliable way to read PSE certificates via RFC.

That knowledge comes from 12 years of staring at SM50, ST22, and STRUST. The AI amplifies it into production code.

### 2. Structure > raw capability

Without the `.claude-docs` system, I'd estimate the project would have taken 12+ months with constant rework. With it, complex features land correctly on the first try because the AI has full context of every architectural decision, every gotcha, and every standard.

### 3. AI verification should be cross-model

No single AI catches everything. Codex found SSRF vulnerabilities Claude missed. Gemini identified connection pool tuning Claude hadn't considered. The combination is stronger than any individual model.

### 4. The methodology is transferable

This isn't specific to monitoring software or SAP. The `.claude-docs` pattern works for any complex project where:
- Development spans weeks or months
- The codebase is too large for AI context windows
- Quality requirements are high
- Domain expertise matters more than coding patterns

## The Result

160+ development sessions. 524 passing tests. Zero production outages. A platform that monitors 26 hosts, 20 resources, and counting — with the architecture to scale to thousands.

The full portfolio (architecture docs, methodology, screenshots, demo video) is at:
**[github.com/josh-lans/monlite-portfolio](https://github.com/josh-lans/monlite-portfolio)**

---

*If you're building something complex with AI assistance and want to discuss the methodology, reach out — joshlans@me.com or [LinkedIn](https://linkedin.com/in/joshlans).*
