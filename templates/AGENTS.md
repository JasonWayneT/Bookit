# Agent Operating Rules for Spec Driven Development

This repository uses BMAD-informed Spec Driven Development. All AI agents must follow this file before making changes.

## Prime directive

Do not edit code first. Every material change must begin with the documentation chain:

1. Review the project constitution.
2. Review BMAD intake and existing requirements.
3. Identify or create the relevant requirement IDs.
4. Update feature, design, test, bug, or change specs as needed.
5. Update the traceability matrix.
6. Create or update implementation tasks.
7. Modify code.
8. Run verification.
9. Update docs with results and unresolved issues.

If the requested change is tiny, use lightweight mode, but still maintain requirement IDs and traceability.

## Required reading order

Before coding, read these files in order when they exist:

1. `docs/spec/00-project-constitution.md`
2. `docs/spec/01-bmad-intake.md`
3. `docs/spec/02-requirements-registry.md`
4. Relevant files under `docs/spec/03-feature-specs/`
5. Relevant files under `docs/spec/04-design-specs/`
6. Relevant files under `docs/spec/05-change-requests/`
7. `docs/spec/06-traceability/traceability-matrix.md`
8. Relevant ADRs under `docs/spec/07-decisions/`
9. Relevant test specs under `docs/spec/08-test-specs/`
10. Relevant known issues under `docs/spec/09-known-issues/`

If a file does not exist because the project is lightweight, continue with the available files.

## Requirement ID rule

Every code change must cite at least one requirement, acceptance criterion, bug ID, or ADR in the task description, commit message, PR summary, or implementation notes.

Preferred ID namespaces:

- `FR-*`: Functional requirement
- `NFR-*`: Non-functional requirement
- `UX-*`: User experience requirement
- `DES-*`: Visual or interaction design requirement
- `ARCH-*`: Architecture requirement
- `DATA-*`: Data, schema, retention, import, or export requirement
- `SEC-*`: Security, privacy, auth, permissions, or compliance requirement
- `INT-*`: Integration requirement
- `OPS-*`: Deployment, observability, reliability, or operations requirement
- `AC-*`: Acceptance criterion
- `TASK-*`: Implementation task
- `TEST-*`: Test case or verification requirement
- `BUG-*`: Known bug, regression, or defect
- `ADR-*`: Architecture decision record
- `CR-*`: Change request

Projects may add namespaces, but must not reuse an ID for a different meaning.

## Change workflow

When the user asks for a change:

1. Restate the requested change in implementation-neutral language.
2. Determine whether this is a feature, bug fix, refactor, design change, architecture change, or documentation-only change.
3. Create or update a `CR-*` entry.
4. Identify affected requirement IDs.
5. If no requirement exists, propose a new requirement ID.
6. Update acceptance criteria.
7. Update design, architecture, UX, data, security, and test specs only if affected.
8. Update the traceability matrix.
9. Produce an implementation task list.
10. Implement the smallest safe change.
11. Run relevant tests or explain why tests could not be run.
12. Update the change request with the verification result.

## Rebuild-from-scratch workflow

If the codebase is lost or must be regenerated:

1. Treat `docs/spec/` as the source of truth.
2. Review all `BUG-*` records so known bugs are not reintroduced.
3. Review all accepted `ADR-*` decisions.
4. Generate implementation tasks from the traceability matrix.
5. Implement requirements in priority order.
6. Validate each `AC-*` and `TEST-*`.
7. Record any behavior that differs from the previous implementation.

## Bug fix workflow

For bugs:

1. Create or update a `BUG-*` record.
2. Link it to affected `FR-*`, `NFR-*`, `UX-*`, `DATA-*`, `SEC-*`, or `ARCH-*` IDs.
3. Add a regression acceptance criterion.
4. Add or update a `TEST-*` case.
5. Fix the bug.
6. Verify that the regression test fails before the fix when feasible and passes after the fix.

## Refactor workflow

For refactors:

1. Link the refactor to an `ARCH-*`, `NFR-*`, `OPS-*`, or `ADR-*` ID.
2. State unchanged external behavior.
3. Identify tests that prove behavior did not change.
4. Implement in small steps.
5. Update design docs only if the architecture actually changed.

## Prohibited behavior

Agents must not:

- Invent requirements silently.
- Delete requirement IDs without recording a replacement or deprecation.
- Implement a feature that has no acceptance criteria.
- Refactor across unrelated areas without a linked requirement or ADR.
- Mark a task complete without verification notes.
- Treat generated code as the source of truth when docs disagree.

## Output format for agent work

When finished, report:

- Requirements touched
- Specs updated
- Code files changed
- Tests run
- Traceability updates
- Open questions or risks

