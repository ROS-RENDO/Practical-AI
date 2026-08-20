# Research: Task Search

**Feature**: Task Search (`001-search-tasks`)
**Date**: 2026-08-20
**Spec**: [spec.md](./spec.md)

## Research Questions

This research resolves the open technical questions for the Task Search feature
within the project's existing stack (React + TypeScript frontend, Node.js +
Express backend, PostgreSQL database) while respecting the constitution
principles (simple architecture, no new frameworks unless necessary, secure
input handling).

---

### RQ-1: How should case-insensitive partial-title matching be implemented?

**Decision**: Implement server-side matching in PostgreSQL using `ILIKE` with
`%`-pattern wildcards, escaping literal `%`/`_` characters in the user input.

**Rationale**:
- Case-insensitivity is a hard requirement (FR-002). `ILIKE` is the standard,
  built-in PostgreSQL mechanism — no extensions or new dependencies.
- Works for partial matches ("part or all of a task title").
- Matching on the backend keeps the existing "search by title" contract
  reusable by any client and keeps the frontend thin (constitution IV).

**Alternatives considered**:
- `LOWER(title) LIKE LOWER($1)`: equivalent capability but more verbose; ILIKE
  is the idiomatic form.
- `citext` column type: changes the schema and does not cover partial matching
  without `%` patterns; more invasive than needed.
- Client-side filtering over a full-task fetch: rejected because it requires
  holding all tasks on the client and does not scale; the existing `GET
  /api/tasks` endpoint already supports server-side querying.
- `pg_trgm` / full-text search: rejected as over-engineering for this scope
  (constitution III — start simple; fuzzy/full-text explicitly out of scope in
  the spec).

---

### RQ-2: How should the search endpoint behave for empty, partial, and missing queries?

**Decision**: One endpoint `GET /api/tasks`; accept an optional `q` query
parameter. Absent or blank `q` returns the complete task list (FR-003,
FR-005). A non-blank `q` returns only matching tasks. Trim leading/trailing
whitespace from `q`.

**Rationale**:
- Single endpoint keeps the API surface minimal and matches the existing task
  listing endpoint (reuse, constitution III/IV).
- "Clearing the search restores the complete task list" maps directly to
  "re-request with an empty query", so one endpoint serves both behaviors.
- Trimming avoids accidental non-matches from whitespace edge cases (spec edge
  cases: leading/trailing spaces).

**Alternatives considered**:
- Separate `/api/tasks/search` endpoint: redundant, adds surface area.
- No-query → 400: rejected; the browser-facing behavior (empty search box
  shows full list) requires the "empty query = full list" semantics.

---

### RQ-3: How should rapid typing avoid stale or excessive requests?

**Decision**: Debounce input changes on the frontend (300 ms), cancel/ignore
in-flight responses that are no longer current (request sequence guard), and
only issue a request when the query settles. Requests are sequenced so that the
latest query always wins (FR-006).

**Rationale**:
- Users type quickly; firing a request per keystroke floods the API
  (FR-006 / spec edge case "user types very quickly").
- A sequence guard (tagging the latest request and discarding stale responses)
  guarantees stale results never overwrite newer ones.
- 300 ms is the standard interactive-search debounce; it keeps the perceived
  latency under the SC-001 target (<2 seconds) by a wide margin.

**Alternatives considered**:
- No debounce (request per keystroke): simple but wasteful and risks stale
  responses racing.
- Long debounce (500 ms+): reduces requests further but adds perceived lag;
  unnecessary at this scale.
- Server-side only (no debounce): still leaves the client spam problem.

---

### RQ-4: How should errors and the "no results" case be handled across layers?

**Decision**: 
- Backend: return `200` with an empty `tasks` array when nothing matches; return
  a structured `5xx` JSON error if the search itself fails.
- Frontend: on empty results show a clear "no matching tasks" message; on error
  show a clear error message, keep the last successful task list visible, and
  do not wipe the list (FR-004, FR-007).

**Rationale**:
- Empty matches are a legitimate, expected outcome — a `200` with `[]` avoids
  treating "no results" as an error (spec User Story 3).
- Keeping the last valid list on error preserves user context instead of
  replacing data with a blank/empty state (FR-007).

**Alternatives considered**:
- `404` for no matches: wrong semantics — the resource (search) succeeded.
- Clearing the list on error: violates FR-007; rejected.

---

### RQ-5: How should the search term be validated and secured?

**Decision**: 
- Treat `q` as untrusted input. Use parameterized queries for all SQL (no
  string concatenation), so the input cannot alter the query structure.
- Escape `%` and `_` within the term so they match literally rather than acting
  as `LIKE` wildcards.
- Enforce a maximum length on `q` (e.g., 200 characters) to bound query cost.

**Rationale**:
- Parameterized queries are the baseline defense against SQL injection
  (constitution V — input validation and error handling).
- Escaping wildcards prevents a user typing `100%` from inadvertently matching
  all titles; it preserves "match the intended text" (spec edge cases).
- Length bounding keeps worst-case work predictable on the database.

**Alternatives considered**:
- Allow raw wildcards (no escaping): rejected — surprising results and a mild
  DoS surface.
- Full-text materialization: over-engineering for this scope (constitution III).

---

### RQ-6: What should the plan's source layout look like under the existing structure?

**Decision**: Follow the existing application layout inferred from the stack
(frontend `*/src/components` + `*/src/services`, backend `*/src/routes` +
`*/src/db`), adding only the files required for this feature. No new structure
or framework is introduced.

**Rationale**:
- Constitution IV requires readable, maintainable code consistent with the
  codebase; introducing new directories or abstractions would add unnecessary
  complexity (constitution III).
- Reuse existing task list components, the existing route/controller pattern,
  and the existing test setup (user directive: "Reuse existing components and
  services where possible").

**Alternatives considered**:
- Feature folders per layer split (feature-flagging modules): rejected —
  over-engineering for a single endpoint + input; the existing structure is
  adequate.