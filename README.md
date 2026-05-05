# BMAD to Spec Driven Development Template

This template turns BMAD planning outputs into a repeatable Spec Driven Development workflow for AI coding agents.

It is designed for projects where requirements, specs, tasks, code, tests, and later changes must stay traceable to durable requirement IDs. It also supports lightweight projects where you only want a small functional spec and a short task list.

## Sources and design influences

- Kiro's public spec documentation describes a three-phase workflow of requirements or bug analysis, design, and implementation tasks, with `requirements.md` or `bugfix.md`, `design.md`, and `tasks.md` as the core files: https://kiro.dev/docs/specs/
- BMAD's public agent documentation shows a workflow-oriented model where analyst, product manager, architect, developer, UX designer, and technical writer agents produce and validate artifacts such as PRDs, epics, stories, architecture, UX design, and dev stories: https://docs.bmad-method.org/reference/agents/
- DeepLearning.AI's public course page frames spec-driven development for coding agents around a project constitution, feature specs, plan-implement-verify loops, and context preservation across agent sessions: https://www.deeplearning.ai/short-courses/spec-driven-development-with-coding-agents/

## What this package gives you

- A project-level documentation structure that coding agents must update before touching code.
- A flexible requirement ID scheme that supports functional, non-functional, design, UX, architecture, data, security, test, bug, and decision records without forcing all categories on every project.
- A BMAD intake process that normalizes PRDs, epics, stories, UX outputs, architecture docs, and dev stories into downstream implementation specs.
- A change workflow for new features, bug fixes, refactors, design updates, and rebuild-from-scratch scenarios.
- Copy-pasteable repo instruction files for `AGENTS.md`, `CLAUDE.md`, `GEMINI.md`, `CURSOR.md`, or similar AI coding environments.
- An optional Agent Skill in `skill/spec-driven-bmad/SKILL.md`.

## Recommended repo structure

```text
docs/
  spec/
    00-project-constitution.md
    01-bmad-intake.md
    02-requirements-registry.md
    03-feature-specs/
      FEAT-001-example.md
    04-design-specs/
      DESIGN-001-example.md
    05-change-requests/
      CR-001-example.md
    06-traceability/
      traceability-matrix.md
    07-decisions/
      ADR-001-example.md
    08-test-specs/
      TEST-001-example.md
    09-known-issues/
      BUG-001-example.md
AGENTS.md
```

For small projects, use only:

```text
docs/spec/00-project-constitution.md
docs/spec/02-requirements-registry.md
docs/spec/03-feature-specs/FEAT-001.md
docs/spec/06-traceability/traceability-matrix.md
AGENTS.md
```

## Operating principle

No agent may implement a code change until it can answer:

1. Which requirement IDs justify this change?
2. Which spec files changed or were reviewed?
3. Which acceptance criteria prove the change is correct?
4. Which tests map to those acceptance criteria?
5. Which known bugs, constraints, or decisions must not be reintroduced?

If the answer is unclear, the agent must update the documentation first or ask for clarification.

## Lightweight, standard, and rigorous modes

### Lightweight mode

Use this for prototypes, scripts, calculators, demos, or very small apps.

Required files:

- `00-project-constitution.md`
- `02-requirements-registry.md`
- One feature spec
- `traceability-matrix.md`
- `AGENTS.md`

Minimum IDs:

- `FR-*` for functional requirements
- `AC-*` for acceptance criteria
- `TASK-*` for implementation tasks
- `TEST-*` for tests

### Standard mode

Use this for normal production apps.

Add:

- `01-bmad-intake.md`
- Design specs
- Change requests
- ADRs
- Known issues

Recommended IDs:

- `FR-*`, `NFR-*`, `UX-*`, `DES-*`, `ARCH-*`, `DATA-*`, `SEC-*`, `AC-*`, `TASK-*`, `TEST-*`, `BUG-*`, `ADR-*`

### Rigorous mode

Use this for enterprise, regulated, mission-critical, multi-agent, or long-lived projects.

Add:

- Requirement status lifecycle
- Explicit owners
- Risk classification
- Compliance and security mappings
- Release gates
- Full bidirectional traceability

## First setup checklist

1. Copy the relevant templates from `templates/` into your repo.
2. Rename `templates/AGENTS.md` to `AGENTS.md`, then duplicate it to `CLAUDE.md`, `GEMINI.md`, or other agent instruction files as needed.
3. Paste BMAD outputs into `docs/spec/01-bmad-intake.md`.
4. Normalize BMAD artifacts into `02-requirements-registry.md`.
5. Create the first feature spec under `03-feature-specs/`.
6. Build or update the traceability matrix.
7. Only then ask an agent to scaffold or change code.

## Change request rule

Every change starts with a change request. Even if you ask an agent informally, the agent must convert the request into:

- A proposed requirement/spec update.
- A list of impacted files.
- A traceability update.
- An implementation task plan.
- A verification plan.

Only after those are consistent may the agent modify code.

