# MonLite Platform Architecture

> Technical overview of MonLite's architecture for engineering portfolio purposes.

## System Overview

MonLite follows a **Company > Platform > System > Host > Connector** hierarchy model. Collectors (distributed Python agents) push metrics via HTTPS to the central control plane. The backend processes, evaluates, and stores metrics while the frontend provides real-time dashboards and AI-powered diagnostics.

**Scale target:** 3,000+ hosts, 200+ concurrent users.

---

## Backend Architecture

### Tech Stack
- **Framework:** FastAPI + SQLAlchemy 2.0 + Pydantic v2 + Uvicorn
- **Database:** PostgreSQL 16 (TIMESTAMPTZ, enforced SSL in production)
- **AI Engine:** Multi-provider LLM (Ollama/Qwen local, Gemini, OpenAI, Anthropic)

### Route Modules (23 files)
The backend is organized into focused route modules:
- `hierarchy.py` — CMDB tree CRUD, dropdown, toggle
- `connections.py` — Node detail, credential/collector inheritance
- `thresholds.py` — 5-level threshold hierarchy with profiles
- `collection.py` — Metric ingest, task scheduling, result processing
- `collectors.py` — Agent registration, pairing, push updates
- `metrics.py` — Time series, rollups, availability, charts, SDT helpers
- `alarms.py` — CRUD, acknowledge, snooze, resolve, statistics
- `luna.py` — AI chat with RAG pipeline + admin controls
- `auth.py` — Login, HttpOnly cookie sessions, OIDC SSO
- `sap_connections.py` — SAP connector CRUD, surveillance rules, seed rules
- And 12 more (users, teams, credentials, integrations, runbooks, SDT, webhooks, etc.)

### Service Layer
- **Metric Aggregation** — Interval selection, time bucketing, SDT-aware helpers
- **Threshold Resolution** — 5-level walk-up (Endpoint > Host > System > Platform > Company > Profile > Global)
- **Stability Scoring** — 0-100 composite score per host from metric rollups (volatility, stress, trend)
- **Predictive Storage** — Linear regression on 30-day daily-max data, R²≥0.5 quality gates
- **Backup Service** — pg_dump + config snapshots with selective restore and timezone-aware scheduling

### Key Design Patterns

**Metric Registry** (`metric_registry.py`): Plugin registry per profile type (linux, windows, web_service, + 7 database engines + SAP). Single source of truth for thresholds, intervals, timeouts, alarm metrics, and metric groups. Drives the entire threshold/alarm pipeline.

**Alarm Engine:** Evaluates metrics against threshold hierarchy with breach counters for consecutive violations. Separate evaluators per domain: host metrics, availability, web service, database, certificate expiry, SAP surveillance rules, predictive storage. Every alarm carries resource-level context (db_connection_id, endpoint_id, sap_connection_id).

**Batch Query Pattern:** Never query in loops. Pre-fetch into `{id: data}` maps, assemble in Python. Zero N+1 queries across the entire codebase.

**Connection Pool:** `pool_size=100, max_overflow=50, pool_use_lifo=True, pool_pre_ping=True, pool_recycle=3600`.

---

## Frontend Architecture

### Tech Stack
- **Framework:** React 19, Vite 7, Tailwind CSS 4
- **Charts:** ApexCharts, D3.js (constellation visualization)
- **Design Language:** "Glass & Moonlight" — glassmorphism with backdrop blur, starfield animation, dark/light themes

### Component Hierarchy
```
App.jsx (HashRouter, 16 routes) → AppLayout.jsx (Sidebar + content)
Pages: Dashboard, Connections, Thresholds, Alarms, Visualizations, Collectors,
       Credentials, Integrations, Runbooks, Maintenance, Certificates, AuditLogs,
       SystemUpdates, UsersSettings, Help, Login
Floating: LunaWidget (AI chat), ToastContext (notifications)
```

### State Management
Custom hooks (no Redux/Context):
- `useConnectionsState` — 30+ useState, 6 useMemo, all Connections page state
- `useCollectorsState` — 50+ useState, 30 handlers
- Pattern: hook returns state object → passed as `state` prop → children destructure

### Key UI Patterns
- **PopOutCard** — Expand-to-modal wrapper for data cards
- **Progressive disclosure** — Collapsible sections, expand-on-demand detail panels
- **Inline threshold editing** — Edit thresholds directly on connection detail panels
- **Infrastructure Constellation** — D3 force-directed visualization with orbital layout, collector group nebulae, real-time alarm coloring

---

## Collector Architecture

### Design
Distributed Python agents that run at the edge (on-premise, behind firewalls):
- **Deployment:** Docker containers or systemd services
- **Communication:** HTTPS push to control plane (no inbound ports required)
- **Parallelism:** ThreadPoolExecutor (10 DB workers, 15 WS workers, 20 host workers)
- **Auto-update:** Remote push updates from the UI (no SSH required)
- **TLS pinning:** Support for certificate pinning to control plane

### Collection Pipeline
```
1. Agent registers via pairing code
2. Heartbeat loop (30s) — reports health, receives config
3. Pending task poll — backend schedules what's due
4. Parallel execution — each check type in its own worker
5. Result push — metrics, alarms, system info sent to backend
```

### Supported Protocols
| Protocol | Library | Use Case |
|----------|---------|----------|
| SSH | paramiko | Linux host metrics |
| WinRM | pywinrm | Windows host metrics |
| SNMP v2c/v3 | pysnmp | Network device metrics |
| HTTP/HTTPS | requests | Web service monitoring |
| MSSQL | pymssql | SQL Server metrics |
| Oracle | oracledb | Oracle metrics |
| SAP HANA | hdbcli | HANA metrics (3-strategy NAT cascade) |
| IBM DB2 | ibm_db | DB2 metrics |
| Sybase ASE | pyodbc + FreeTDS | Sybase metrics |
| SAP MaxDB | jpype1 + sapdbc.jar | MaxDB metrics (JVM bridge) |
| SAP RFC | pyrfc / JCo via jpype1 | ABAP application monitoring |
| SOAP/HTTP | requests | SAPControl monitoring |
| ICMP/TCP | socket | Availability ping |

---

## Alarm System

### Threshold Hierarchy (6 levels)
```
Endpoint/Connector → Host → System → Platform → Company → Profile → Global
```
Most specific wins. Profiles allow grouping (e.g., "Production SAP" profile applied to a platform).

### Breach Counters
Alarms don't fire on a single threshold violation. Configurable `trigger_polls` (consecutive breaches before alarm) and `clear_polls` (consecutive clears before auto-resolve). Default: 3 trigger / 2 clear for performance metrics, 1 trigger for availability.

### Surveillance Rules Engine (SAP)
For SAP monitoring, simple thresholds aren't sufficient. MonLite uses `SapSurveillanceRule` with:
- Per-entity evaluation (each WP, job, lock = own alarm)
- Regex filtering (names, users, programs)
- Aggregate vs individual counting modes
- Percentage-based thresholds
- Runtime duration thresholds
- Exclusion rules (suppress known-safe entities)

### Remediation Hints
Every alarm type has registered remediation templates in `remediation_registry.py`:
- Pattern-based matching (regex on error messages)
- Template variables: `{resource_name}`, `{host_name}`, `{value}`, `{threshold}`, `{severity}`
- 100+ alarm types covered across all connector types

### Webhook Queue
Persistent outbound queue (not fire-and-forget):
- Exponential backoff: 2s → 4s → 16s → 60s → 300s
- Dead letter after 5 attempts
- Per-alarm sequential ordering (create before clear)
- Routing rules with include/exclude conditions
- 44 template variables for payload customization
- 6 presets: ServiceNow ITOM, ServiceNow Incident, Slack, Teams, generic

---

## AI Assistant — Luna

### Architecture
- **Intent Detection:** Pattern-based intent classification (16 intents)
- **Data Injection:** Real-time database queries scoped by RBAC
- **Cross-Correlation:** Auto-links related intents (e.g., database → storage → alarms)
- **Fabrication Validator:** Extracts ground truth from context, redacts hallucinated values
- **RAG Pipeline:** Indexes 27 help documentation pages for contextual answers

### Supported Providers
| Provider | Deployment | Use Case |
|----------|-----------|----------|
| Ollama (Qwen 2.5) | Local, air-gapped | Production — no data leaves firewall |
| Google Gemini | Cloud API | Enhanced analysis, faster responses |
| OpenAI | Cloud API | Alternative cloud provider |
| Anthropic | Cloud API | Alternative cloud provider |

---

## Security Architecture

### Authentication
- **Local accounts:** Argon2-hashed passwords, configurable password policy, account lockout
- **LDAP:** Active Directory integration
- **SSO:** OIDC Authorization Code + PKCE (Okta, Microsoft Entra ID) with JIT provisioning, group-to-role mapping, Force SSO mode

### Session Security
- HttpOnly cookies (no localStorage tokens)
- CSRF double-submit protection
- Configurable inactivity timeout + session max lifetime
- Concurrent session limits
- Session write throttling (~98% fewer DB writes)

### Data Protection
- Fernet encryption (AES-128-CBC + HMAC) for credentials at rest
- HMAC-signed audit logs with integrity verification
- Enforced PostgreSQL SSL in production
- 0600 key file permissions

### Application Hardening
Independent security audit (Codex P0) covering:
- CSP + Permissions-Policy headers
- XSS sanitization via DOMPurify
- SSRF fail-closed on DNS errors
- CORS hardening
- ZIP path traversal guards
- Rate limiting fail-closed in production
- DataTable row caps (DOM bloat prevention)

---

## Database Monitoring — Deep Dive

### Per-Engine Specialization

**SAP HANA** — 3-strategy connection cascade:
1. Direct TCP connection
2. Docker port-mapped connection (NAT traversal)
3. SYSTEMDB connection with tenant routing

**Oracle** — System view queries with tablespace-level granularity, SGA/PGA monitoring, redo log tracking.

**MSSQL** — Backup age monitoring with status tracking, PLE (Page Life Expectancy), batch request rates.

**IBM DB2** — Bufferpool hit ratio decomposition, log utilization, Community and Enterprise edition support.

**SAP MaxDB** — JVM bridge (jpype1 + sapdbc.jar) for JDBC connectivity. No native Python driver exists.

### Smart Storage Monitoring
- Autogrow-aware capacity calculation (MSSQL)
- Per-tablespace thresholds (Oracle, HANA)
- Predictive days-to-full projections (linear regression, 14-day minimum data, R²≥0.5)

---

## Scalability Patterns

| Pattern | Implementation |
|---------|---------------|
| **ETag caching** | 304 Not Modified across all API endpoints |
| **SQL subquery RBAC** | No post-fetch filtering — RBAC applied in SQL WHERE |
| **Metric rollups** | Hourly avg/max aggregation, 90-day retention |
| **Connection pooling** | 100 + 50 overflow, LIFO, pre-ping, 1-hour recycle |
| **Paginated responses** | Default 100, max 5000, `{items, total, limit, offset}` |
| **Distributed collectors** | Edge agents behind firewalls, HTTPS push only |
| **Time-windowed SAP queries** | check_period = interval × 1.5, row limits as safety net |
| **Webhook queue** | Persistent with exponential backoff, not fire-and-forget |
