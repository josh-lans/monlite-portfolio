# Lessons Learned

> **Purpose:** Real mistakes that caused rework, documented to prevent recurrence.
> Every entry here saved hours of debugging the second time around.
>
> **Format:** L-NNN: Short title — explanation of what went wrong and the correct approach.

---

## General Development

**L-001: Read before write.** Never start coding without reading MEMORY.md and checking git state. Stale context = wrong decisions.

**L-002: Test before commit.** "It should work" is not the same as "it does work." Run the test suite.

**L-003: Don't assume — verify.** When debugging, check the actual data (database, API response, network tab) instead of reasoning about what "should" be there.

---

## Architecture

<!-- Add entries as you discover patterns. Examples: -->

<!-- **L-010: Feature X requires wiring in 7 places.** We added the metric but forgot the alert evaluation. Checklist now in DEPENDENCIES.md §D6. -->

<!-- **L-011: The ORM doesn't auto-add columns.** ALTER TABLE migration needed for existing databases. create_all() only works for new tables. -->

---

## Frontend

<!-- **L-020: Component X re-renders on every state change.** Added useMemo. Performance went from 2s to 50ms. -->

---

## Infrastructure

<!-- **L-030: The connection pool exhausts under load.** Increased pool_size from 5 to 20, added pool_pre_ping. -->

---

<!--
Tips:
- Write entries IMMEDIATELY when a mistake causes rework
- Be specific: what happened, why, what's the correct approach
- Number entries (L-NNN) so they can be referenced in STANDARDS.md
- The AI reads this file — entries like "FIELD_NOT_VALID usually means buffer overflow"
  prevent hours of misdiagnosis in future sessions
-->
