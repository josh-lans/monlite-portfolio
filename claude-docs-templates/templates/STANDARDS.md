# Development Standards

> **Purpose:** Mandatory patterns for all development. Referenced by § number.
> **Last Updated:** YYYY-MM-DD

---

## Security & API

**§1 Authentication:** Every endpoint MUST enforce authentication. No unauthenticated access except login/health.

**§2 Authorization:** Every list endpoint MUST filter server-side by user permissions. Never filter client-side.

**§3 Input Validation:** Use Pydantic schemas with `extra="forbid"` on request models. Response models use `from_attributes=True`.

**§4 Pagination:** All list endpoints return `{items, total, limit, offset}`. Default limit=100, max=5000.

---

## Data Patterns

**§5 Batch Queries:** Never query in loops. Pattern: fetch entity list -> collect IDs -> `WHERE id IN (ids)` -> build `{id: data}` maps -> assemble in Python.

**§6 Timestamps:** All timestamp columns use timezone-aware types. Store via `datetime.now(timezone.utc)`. Never store as strings.

**§7 Version Bumps:** Bump component version on ANY change to component files. This is non-negotiable.

---

## Feature Wiring

**§8 New Feature Checklist:**
1. Backend model/migration
2. API endpoint + Pydantic schemas
3. Frontend component
4. Tests
5. Documentation
6. [Add your project-specific wiring steps]

**§9 New Metric/Alert Checklist:**
1. Define in registry/config
2. Collect in data pipeline
3. Store in database
4. Evaluate in alert engine
5. Add remediation hint
6. Add to test suite
7. Add to documentation

---

## Frontend Patterns

**§10 State Management:** [Describe your state management pattern]

**§11 Component Naming:** [Describe your naming conventions]

**§12 Error Handling:** [Describe how errors are displayed to users]

---

## Testing

**§13 Test Before Commit:** Run test suite before every commit. No exceptions.

**§14 Test Coverage:** New features MUST include tests for happy path + error path.

---

<!-- Add more standards as patterns are established. Number everything with §.
     The AI will reference these: "per §5, I'll batch-fetch instead of querying in a loop" -->
