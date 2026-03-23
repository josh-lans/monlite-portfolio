# SAP Monitoring Architecture — Design Overview

## Connector Types

MonLite supports three SAP connector types:

| Type | Protocol | Use Case |
|------|----------|----------|
| **SAPControl** | SOAP/HTTP | OS-level access, process health, WP details, dispatcher queues |
| **ABAP RFC** | SAP RFC (PyRFC/JCo) | Application-level, table reads, RFC function modules, surveillance rules |
| **Unified** | Both | Full coverage — best check from each wins, other auto-disabled |

## ABAP RFC Check Architecture

Each surveillance category follows this pattern:

```
1. Scheduler resolves effective interval (user override → hierarchy → default)
2. Backend dispatches check group with check_period = interval × 1.5
3. Collector calls SAP RFC (FM or RFC_READ_TABLE) with time-windowed WHERE clause
4. Results parsed into entity list (each WP, job, lock, transport = one entity)
5. Alarm engine evaluates SapSurveillanceRule against each entity
6. Matching entities → alarm creation/update with remediation hint
7. Metrics stored for trending/visualization
```

### Time-Windowing Strategy

Every table scan includes a time filter to prevent reading entire tables:
- `check_period = collection_interval × 1.5` (50% overlap buffer for timing drift)
- WHERE clause uses SAP date/time fields (ALDATE, TRTIME, CREDAT, etc.)
- Row limits (ROWCOUNT 300-500) as safety net

### Auto-Detection

- **BW/BI systems**: Process chain (RDA) checks auto-detect SAP_BW/DW4CORE components
- **Certificate tables**: Falls back through STRUSTCERT → SSFCERTST → INST_EXECUTE_REPORT
- **Table field discovery**: If RFC_READ_TABLE returns FIELD_NOT_VALID, auto-discovers valid fields via NO_DATA query

## Surveillance Rules Engine

Unlike simple "metric > threshold → alert", MonLite uses a rules engine:

```
SapSurveillanceRule:
  - name, check_category, sort_order
  - wp_type_filter (DIA|BGD|UPD|SPO|ENQ)
  - wp_state_filter (Running|Stopped|On Hold|PRIV)
  - wp_user_filter, wp_program_filter (regex)
  - count_threshold, count_mode (absolute|percentage)
  - runtime_threshold_seconds
  - severity (warning|major|critical)
  - aggregate_mode (individual|grouped)
  - auto_clear, alarm_enabled
  - exclusion support (is_exclusive flag)
```

This enables:
- "Alert if >25% of DIA WPs are Stopped" (percentage mode + state filter)
- "Alert if any BGD WP runs >24 hours" (runtime mode + type filter)
- "Exclude lock objects matching BGRFC_*" (exclusion + regex)
- "Alert on IDoc errors >5 in aggregate" (grouped count mode)

## Change Settings Audit Trail

Unique feature for compliance/audit:
1. Collector reads T000 (SCC4 client settings) + TMW_GET_SYSTEM_CHANGEABILITY (SE06)
2. Backend compares new snapshot against previous
3. Changed fields inserted into `SapChangeSettingsHistory` with old/new values
4. Frontend shows timeline with translated human-readable labels
5. AI assistant (Luna) has full context for natural-language audit queries

## Design Principles

1. **Never scan full tables** — always time-window or status-filter
2. **Per-entity alarms** — each WP, job, lock gets its own alarm (no aggregation hiding individual issues)
3. **SAP terminology preserved** — Basis admins see exactly what SM50/SM37/ST22 shows
4. **Graceful degradation** — if an FM doesn't exist or a table has different fields, fall back don't crash
5. **Collector-side efficiency** — keep RFC calls short, release WPs quickly, cache negative results (e.g., "not a BW system")
