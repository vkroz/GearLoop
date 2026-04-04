---
status: backlog
type: task
phase: 1
depends_on: [20260404_02_extend_task_yaml_schema]
---

# Add structural validation hooks

## Problem

Every structural guarantee in the plugin (task YAML validity, directory-status consistency, required doc sections) relies on the agent following skill instructions. There is no enforcement layer. The agent can produce structurally invalid artifacts without any mechanism catching it.

## Scope

**In scope:**
- Pre-commit hook that validates:
  - Task files in `tasks/` have valid YAML headers with required fields (`status` at minimum; `phase` and `depends_on` for plan-generated tasks)
  - Task file directory matches its `status` value (file in `backlog/` has `status: backlog`, etc.)
  - No obvious secrets staged (`.env`, files matching common secret patterns)
- Hook must pass/fail with clear error messages indicating what is wrong and where

**Out of scope:**
- Semantic validation (is the architecture document good?)
- Dependency graph cycle detection (deferred to MCP v2)
- Hooks on file-write events (start with commit-time only; expand later if needed)
- Template section validation for `docs/` files (deferred — start minimal)

## Constraints

- Hooks must be shell scripts that work cross-platform (macOS, Linux)
- Hooks must not require external dependencies beyond standard Unix tools and a YAML parser (or regex-based extraction for simple fields)
- Hook failure must produce actionable error messages, not cryptic exits

## Acceptance Criteria

- AC1: Pre-commit hook exists and runs automatically on `git commit`
- AC2: Hook rejects commits where task file `status` field does not match containing directory
- AC3: Hook rejects commits where task files lack required `status` field
- AC4: Hook rejects commits staging files matching `.env*` or common secret patterns
- AC5: Hook passes cleanly on valid task files and non-task commits
- AC6: Error messages name the offending file and the specific validation failure

## Quality Gates

- Test with valid task files in each directory — hook passes
- Test with mismatched status/directory — hook rejects with clear message
- Test with missing YAML header — hook rejects
- Test with non-task commit (code only) — hook passes
