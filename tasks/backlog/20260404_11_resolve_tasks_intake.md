---
status: backlog
type: task
phase: 3
depends_on: []
---

# Resolve tasks/intake directory

## Problem

Files exist in `tasks/intake/` (`agents-orchestrator.md`, `jira audit tool.md`) but the task management standard defines only three lifecycle states: backlog, active, done. `intake` is not documented. This creates confusion for contributors and violates the plugin's own standard.

## Scope

**In scope:**
- Decide: is `intake` a valid lifecycle state?
  - If yes: document it in `rules/task-management.md` with semantics, transition rules, and update the spec
  - If no: move files to `tasks/backlog/` (reformatting to match task standard if needed) or remove them if they are obsolete
- Ensure the `tasks/` directory structure matches the documented standard after this task

**Out of scope:**
- Evaluating or executing the content of the intake files
- Creating new lifecycle states beyond what's needed to resolve this inconsistency

## Acceptance Criteria

- AC1: `tasks/intake/` either does not exist, or is documented in the task management standard
- AC2: All files under `tasks/` conform to the documented lifecycle states
- AC3: Task management standard and spec are consistent

## Quality Gates

- `ls tasks/` output matches the states documented in `rules/task-management.md`
