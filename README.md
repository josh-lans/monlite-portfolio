# MonLite — Enterprise Infrastructure Observability Platform

> **Note**: This is a portfolio showcase of architecture, methodology, and design decisions. The full source code is proprietary.

## What is MonLite?

MonLite is an enterprise-grade infrastructure monitoring platform I designed and built to replace a seven-figure commercial SaaS monitoring contract. It provides real-time observability across SAP systems, databases, Linux/Windows hosts, and web services with AI-powered diagnostics.

**Built in under 4 months** using AI-assisted development (Claude Code + structured session methodology).

## Platform Capabilities

| Domain | Coverage |
|--------|----------|
| **SAP ABAP RFC** | 19 surveillance categories — work processes, jobs, short dumps, locks, transports, IDocs, RFC queues, spool, certificates, change settings audit trail, parameter compliance, and more via direct RFC calls |
| **SAP SAPControl** | Process health, work processes, dispatcher queues, enqueue locks, ICM threads via SOAP API |
| **Databases** | MSSQL, Oracle, SAP HANA, IBM DB2, Sybase ASE, SAP MaxDB, PostgreSQL — connectivity, performance, storage, backup monitoring |
| **Hosts** | Linux (SSH) and Windows (WinRM) — CPU, memory, disk, swap, load, filesystem projections |
| **Web Services** | HTTP/HTTPS availability, response time, SSL certificate expiry |
| **AI Diagnostics** | Luna — air-gapped LLM assistant with RAG pipeline (Ollama/Qwen local + Gemini cloud fallback) |

## Architecture

```
                    ┌─────────────────────────────┐
                    │     MonLite Control Plane    │
                    │  FastAPI + React 19 + Vite   │
                    │  PostgreSQL + Elasticsearch  │
                    └──────────────┬───────────────┘
                                   │ HTTPS/WSS
                    ┌──────────────┴───────────────┐
              ┌─────┴─────┐                 ┌──────┴──────┐
              │ Collector  │                 │  Collector  │
              │  (Edge 1)  │                 │  (Edge 2)   │
              │ Python 3.12│                 │ Python 3.12 │
              └─────┬──────┘                 └──────┬──────┘
                    │                                │
        ┌───────────┼───────────┐         ┌──────────┼──────────┐
        │           │           │         │          │          │
    SAP RFC    Database     SSH/WinRM   SAP SOAP   HTTPS    ICMP/TCP
    (PyRFC)    (pymssql)    (paramiko)  (requests) (certs)  (ping)
```

## Technical Stack

- **Backend**: Python 3.12, FastAPI, SQLAlchemy, Uvicorn, PostgreSQL
- **Frontend**: React 19, Vite, Tailwind CSS, ApexCharts, glassmorphism UI
- **Collectors**: Distributed Python agents with auto-update, hot-reload config
- **AI**: Ollama (local Qwen 2.5), Google Gemini API, RAG with help doc indexing
- **Auth**: Microsoft Entra ID / Okta SSO, RBAC, CSRF, encrypted credential vault
- **Integrations**: ServiceNow CMDB + Event Management, Elasticsearch, Webhooks

## AI-Assisted Development Methodology

One of the most interesting aspects of this project is the development methodology. I used Claude Code (Anthropic's CLI agent) as my primary development partner across 160+ structured sessions.

### The `.claude-docs` System

I developed a structured documentation system that maintains project context across AI sessions:

- **MEMORY.md** — Essential working context, environment details, anti-shortcut protocols
- **STANDARDS.md** — 50+ mandatory development patterns with section references
- **DEPENDENCIES.md** — Change impact checklists (e.g., "adding a new metric requires 7 wiring steps")
- **SESSION_LOG.md** — Detailed session history for context continuity
- **LESSONS.md** — Learned patterns to prevent repeated mistakes

This system enabled consistent, high-quality output across months of development without the AI "forgetting" architectural decisions or taking shortcuts.

### Key Insight

The value wasn't "AI wrote my code." The value was:
1. **Domain expertise** (12 years of SAP Basis) provided the *what* and *why*
2. **AI accelerated** the *how* — translating operational knowledge into production code
3. **Structured methodology** maintained quality at speed

## SAP ABAP RFC Monitoring — Deep Dive

The ABAP RFC connector is the most complex module, providing 19 surveillance categories:

| Category | SAP Source | Key Details |
|----------|-----------|-------------|
| Work Processes | TH_WPINFO | Per-WP-type thresholds (DIA/BGD/UPD/SPO), runtime tracking, PRIV detection |
| Jobs | TBTCO | Time-windowed scans, duration/delay/error detection, top offenders |
| Short Dumps | /SDF/GET_DUMP_LOG | Error ID classification, aggregate + individual alerting |
| Locks | ENQUEUE_READ | Lock age tracking, orphan detection, table-level filtering |
| Transports | TPALOG | Return code filtering (RC>=8 errors), time-windowed import history |
| IDocs | EDIDC | Error/waiting status, direction-aware (inbound/outbound) |
| RFC Queues | TRFC_QIN/QOUT_OVERVIEW | Point-in-time queue snapshots, error detection |
| Spool | TSP01 | Error status monitoring, time-windowed |
| Certificates | SSF_ALERT_CERTEXPIRE | PSE certificate expiry via INST_EXECUTE_REPORT |
| Change Settings | TMW + T000 | SE06/SCC4 drift detection with full audit history trail |
| Parameter Compliance | PAHI | Security baseline monitoring (login/*, rdisp/*, icf/*) |
| And 8 more... | Various | SAPconnect, batch input, app logs, audit logs, RFC destinations, etc. |

### Surveillance Rules Engine

Unlike simple threshold-based monitoring, MonLite uses a rules engine (`SapSurveillanceRule`) that supports:
- Per-entity evaluation (each WP, each job, each lock gets its own alarm)
- Regex filtering on names, users, programs
- Aggregate vs individual counting modes
- Percentage-based thresholds
- Runtime duration thresholds
- Exclusion rules (suppress known-safe entities)
- Auto-clear with configurable trigger/clear poll counts

## Screenshots

*See `/screenshots/` directory for UI examples.*

## Design Philosophy

- **Progressive disclosure** — Simple by default, power features on expand
- **SAP terminology preserved** — disp+work, icman, gwrd — no prettifying for Basis admins
- **Glassmorphism UI** — Premium feel with semi-transparent cards, backdrop blur, dark/light modes
- **Air-gapped first** — Luna AI works fully offline; cloud is optional enhancement
- **Scale-aware** — Every RFC call is time-windowed and row-limited to prevent WP exhaustion on production SAP systems

## Contact

**Joshua Lans** — joshlans@me.com — [LinkedIn](https://linkedin.com/in/joshlans)
