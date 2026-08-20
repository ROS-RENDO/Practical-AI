<!--
SYNC IMPACT REPORT
- Version change: (unversioned template) → 1.0.0
- Modified principles: none (initial adoption — all placeholders concretized)
- Added sections: Core Principles (5), Testing & Quality Gates, Development
  Workflow, Governance
- Removed sections: none
- Deferred TODOs: none
-->

# Practical-AI Constitution

## Core Principles

### I. Specification-First (NON-NEGOTIABLE)

Every feature MUST begin with an executable specification — Outcomes,
Boundaries, Constraints, Prior Decisions, Task Breakdown, Verification
Criteria — written before any implementation code. The specification is the
source of truth: implementation MUST be driven by the spec, and the spec MUST
NOT be rewritten to match code. Work MUST NOT jump directly from an idea to
implementation.

### II. Verifiable Completion

No implementation MAY be considered complete unless its acceptance criteria
can be verified. Every Verification Criterion MUST be checked against the
actual output, pass or fail, with evidence. "Looks right" MUST NOT be accepted
as equivalent to "passes verification."

### III. Simple Architecture

The simplest design that satisfies the specification MUST be chosen. Unnecessary
complexity, speculative generality, over-abstraction, and new frameworks MUST
NOT be introduced without a stated justification. Each component MUST have
separation of concerns and a single, clear responsibility (YAGNI).

### IV. Readable & Maintainable Code

Code MUST be readable and maintainable: clear naming, consistent structure, and
small focused units. Important architectural decisions MUST be documented.
Reviews MUST treat readability as a blocking concern, not a preference.

### V. Security, Input Validation & Error Handling

Secrets MUST be managed through environment configuration and MUST NEVER be
committed to version control. All inputs MUST be validated. Errors MUST be
handled gracefully, including clear fallbacks when external services (e.g., AI
providers) fail or return unusable output.

## Testing & Quality Gates

Automated tests MUST cover important business logic, especially the acceptance
criteria defined in the specification. New behavior MUST be specified before
tests are written, and critical logic MUST be verified by tests that fail before
the implementation exists (Red-Green-Refactor). A Quality Gate MUST be passed
before implementation begins: the spec is clear, ambiguities are resolved, the
plan satisfies the spec, tasks implement the plan, and `/speckit.analyze`
reports no unresolved inconsistencies.

## Development Workflow

Features MUST move through the SDD pipeline in order: Specify → Clarify → Plan →
Tasks → Analyze → Implement → Validate. Traceability MUST be maintained end to
end: every requirement MUST be represented in the plan, implemented by a task,
and verified by a test or check. Before implementation, consistency between
specification, plan, and tasks MUST be confirmed. After implementation, the
result MUST be validated against the original specification, and any drift MUST
be recorded and corrected.

## Governance

This Constitution supersedes all other practices where they conflict. Amendments
MUST be documented in this file with a semantic version increment
(MAJOR.MINOR.PATCH) and an updated Last Amended date. Compliance MUST be
reviewed at the end of every feature: the implementation is checked against the
specification, and results are recorded. Every pull request and review MUST
verify constitution compliance; complexity MUST be justified in review.

**Version**: 1.0.0 | **Ratified**: 2026-08-20 | **Last Amended**: 2026-08-20
