# Implementation Plan: Task Search

**Branch**: `001-search-tasks` | **Date**: 2026-08-20 | **Spec**: [spec.md](./spec.md)

**Input**: Feature specification from `/specs/001-search-tasks/spec.md`

## Summary

Users must be able to find a specific task by typing part or all of its title,
with results updating as they type. Search matches case-insensitively on the
title (FR-002), the full task list is shown when the search is empty or cleared
(FR-003, FR-005), a clear "no results" message appears when nothing matches
(FR-004), stale results must never overwrite newer ones (FR-006), and errors
must show a clear message while keeping the last valid list (FR-007). The
approach reuses the existing task API (`GET /api/tasks` now accepts an optional
`q` parameter) and the existing task list UI; no new frameworks are introduced.

## Technical Context

**Language/Version**: TypeScript. Frontend React + TypeScript; backend Node.js +
Express.

**Primary Dependencies**: React (frontend), Express (backend), PostgreSQL driver
used by the existing backend (reuse the project's current database access
pattern).

**Storage**: PostgreSQL. The existing `tasks` table already contains the `title`
column; no schema change is required.

**Testing**: Reuse the project's current testing approach. Backend HTTP/integration
tests (+ unit tests for the query builder); frontend component tests for search
behavior. Exact framework follows the existing repository (research.md RQ-6).

**Target Platform**: Web application (browser frontend, Node.js backend).

**Project Type**: Web application — separate frontend and backend.

**Performance Goals**: Search results must arrive such that a known task is found
within 2 seconds of the first keystroke (SC-001). 300 ms input debounce is the
interaction target (research RQ-3).

**Constraints**: No new frameworks beyond the existing React + Express +
PostgreSQL stack (user directive); reuse existing components/services; follow the
project's current folder structure and testing approach.

**Scale/Scope**: Single team capstone; task counts in the low thousands. ILIKE
substring search on an indexed title column is sufficient; no search engine.

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| Principle | Compliance | Note |
|-----------|------------|------|
| I. Specification-First | PASS | Implementation begins only after spec, plan, tasks exist |
| II. Verifiable Completion | PASS | Every FR and SC maps to a checkable test/acceptance scenario |
| III. Simple Architecture | PASS | Single existing endpoint reused; ILIKE search; no new framework |
| IV. Readable & Maintainable | PASS | Follows existing folder structure and code conventions |
| V. Security, Input Validation & Error Handling | PASS | Parameterized queries, literal `%`/`_` escaping, trimmed and length-bounded input, graceful error fallback (FR-007) |

No violations require Complexity Tracking.

| Gate | Status |
|------|--------|
| Spec clear and unambiguous | PASS |
| Plan satisfies specification | PASS |
| Tasks will implement plan | PASS |
| `/speckit.analyze` reports no unresolved inconsistencies | Pending (run before implementation) |

## Project Structure

### Documentation (this feature)

```text
specs/001-search-tasks/
├── plan.md              # This file (/speckit.plan command output)
├── research.md          # Phase 0 output (/speckit.plan command)
├── data-model.md        # Phase 1 output (/speckit.plan command)
├── quickstart.md        # Phase 1 output (/speckit.plan command)
├── contracts/           # Phase 1 output (/speckit.plan command)
└── tasks.md             # Phase 2 output (/speckit.tasks command)
```

### Source Code (repository root)

```text
# Web application: separate frontend and backend (existing structure)
backend/
├── src/
│   ├── routes/
│   │   └── tasks.ts            # GET /api/tasks (adds ?q= search support)
│   └── db/tasks.ts             # query builder / data access for tasks
├── tests/
│   ├── contract/tasks.test.ts  # contract tests for GET /api/tasks
│   └── api/tasks.test.ts       # endpoint behavior: search, empty, error
└── package.json

frontend/
├── src/
│   ├── components/
│   │   ├── TaskSearch.tsx      # controlled search input (300ms debounce)
│   │   └── TaskList.tsx        # existing task list (reused)
│   └── services/
│       └── taskApi.ts          # API client; adds searchQuery()
└── tests/
    └── components/TaskSearch.test.tsx
```

**Structure Decision**: Follow the existing web-app layout (`backend/` +
`frontend/` with `src/routes`, `src/components`, `src/services`). New files are
added only where the feature requires them — a search input component, an API
client search method, and the backend route/query changes. No new directories
or abstractions are introduced (constitution III, IV).

## Complexity Tracking

> Not required — no Constitution Check violations.