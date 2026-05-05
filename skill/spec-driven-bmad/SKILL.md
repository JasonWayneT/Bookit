---
name: spec-driven-bmad
description: "Use when the user wants BMAD outputs, PRDs, epics, stories, architecture docs, UX notes, or change requests converted into Spec Driven Development artifacts before coding. Enforces requirement IDs, traceability, change-first documentation, and agent coding rules."
license: MIT
metadata:
  version: '1.0'
---

# Spec Driven BMAD

## When to use this skill

Use this skill when the user wants to:

- Scaffold a new project from BMAD documentation.
- Convert BMAD outputs into implementation-ready specs.
- Add AI coding rules that require specs before code.
- Maintain requirement IDs and traceability across code, tests, and documentation.
- Make a project rebuildable from specs if the codebase is lost.
- Add a repeatable workflow for feature changes, bug fixes, refactors, or design updates.

## Core principle

The documentation chain is the source of truth. Code is an implementation of accepted requirements, specs, decisions, and tests.

## Required workflow

1. Determine project mode: lightweight, standard, or rigorous.
2. Read or create the project constitution.
3. Import BMAD outputs into a BMAD intake file.
4. Normalize the BMAD outputs into a requirements registry.
5. Assign stable IDs to requirements, acceptance criteria, tasks, tests, bugs, and decisions.
6. Create or update feature specs and design specs.
7. Update the traceability matrix.
8. Create implementation tasks that cite requirement IDs.
9. Only then modify code or instruct a coding agent to modify code.
10. Verify against acceptance criteria and update the traceability matrix.

## ID namespaces

Use only the categories the project needs:

- `FR-*`: Functional requirement
- `NFR-*`: Non-functional requirement
- `UX-*`: User experience requirement
- `DES-*`: Visual or interaction design requirement
- `ARCH-*`: Architecture requirement
- `DATA-*`: Data requirement
- `SEC-*`: Security or privacy requirement
- `INT-*`: Integration requirement
- `OPS-*`: Operations requirement
- `AC-*`: Acceptance criterion
- `TASK-*`: Implementation task
- `TEST-*`: Test case
- `BUG-*`: Known bug or regression
- `ADR-*`: Architecture decision record
- `CR-*`: Change request

## Change-first rule

When the user asks for a change, do not code immediately. First:

1. Create or update a `CR-*`.
2. Link affected requirements.
3. Add or update acceptance criteria.
4. Update design, test, bug, or ADR docs if impacted.
5. Update traceability.
6. Then implement.

## Lightweight mode

For small apps, require only:

- Project constitution
- Requirements registry
- One feature spec
- Traceability matrix
- Agent instruction file

Minimum IDs:

- `FR-*`
- `AC-*`
- `TASK-*`
- `TEST-*`

## Final response format

When work is complete, summarize:

- Requirements touched
- Specs updated
- Code files changed
- Tests run
- Traceability updates
- Open questions or risks

