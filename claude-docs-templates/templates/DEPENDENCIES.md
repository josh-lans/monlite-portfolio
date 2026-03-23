# Change Impact — Dependency Checklists

> **Purpose:** After any change, scan for matching scenarios and verify all propagation points.
>
> **TOP REMINDER:** If you touched ANY component files, BUMP the version number (§7).

---

## §D1. Add a New API Endpoint
**Files:** routes/*.py (route + schemas + auth) -> main.py (register if new router) -> frontend API calls -> tests -> documentation

## §D2. Add a New Database Model
**Files:** models.py (columns + indexes) -> migration script (ALTER TABLE) -> seed data -> routes -> frontend -> tests
**Rule:** Schema migration tools (Alembic, etc.) don't add columns to existing tables with `create_all()`.

## §D3. Add a New Feature End-to-End
**Files:**
1. Backend model + migration
2. API endpoint + Pydantic schemas
3. Frontend component + routing
4. State management hook/store
5. Tests (backend + frontend)
6. Documentation (help pages, README)
7. [Project-specific: alert wiring, webhook templates, etc.]

## §D4. Update Documentation
**Pipeline:** Edit source docs -> build/copy to served location -> restart backend if cached -> verify in browser

## §D5. Add a Python/JS Dependency
**Files:** requirements.txt, package.json, Dockerfile, install scripts, CI/CD pipeline

## §D6. Add a New Alert/Notification Type
**Files:**
1. Define alert metric in registry
2. Implement evaluation logic in alert engine
3. Add remediation hint template
4. Add to notification dispatch (email, webhook, etc.)
5. Frontend display in alerts page
6. Test: trigger alert, verify notification, verify resolution

## §D7. Add a New Connector/Integration Type
**Files:**
1. Backend model + API routes
2. Collection/polling logic
3. Frontend configuration UI
4. Default settings/thresholds
5. Help documentation
6. **Audit log hierarchy resolution** (MANDATORY — see §D8)
7. Tests

## §D8. Audit Log Hierarchy (MANDATORY for new connectors)
**Where:** Audit logging function — the auto-resolve block.
**Pattern:** Connector models have a parent reference (e.g., host_id). Auto-resolve uses this to populate hierarchy names (company/platform/system/host).
**Checklist:**
1. Add resource_type string to the auto-resolve guard
2. Add lookup logic to resolve parent reference
3. Verify hierarchy filters work on audit logs page
4. Test: create/update/delete the connector, check audit log shows full hierarchy

---

<!-- Add more dependency checklists as you discover propagation patterns.
     The most valuable entries come from bugs: "we changed X but forgot Y" -->
