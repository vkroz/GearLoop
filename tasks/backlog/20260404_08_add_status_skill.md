---
status: backlog
type: task
phase: 3
depends_on: [20260404_02_extend_task_yaml_schema]
---

# Add /stc:status skill

## Problem

No way to detect partial workflow state after a session interruption. The user must manually inspect `docs/` and `tasks/` to figure out where things stopped. Multi-session execution is manual and error-prone.

## Scope

**In scope:**
- New skill: `skills/status/SKILL.md`
- Scans `docs/` for artifact presence to determine which SDLC stages are complete
- Scans `tasks/` subdirectories to report task counts by status (backlog, active, done)
- Reports current implementation progress (active phase, blocked tasks, next expected action)
- Flags inconsistencies (e.g., task marked active but no recent changes, docs missing for a stage that should be complete)
- Update `README.md` and spec to document the new skill

**Out of scope:**
- Modifying workflow state (read-only skill)
- Dependency graph visualization
- Suggesting which skill to run next beyond stating the expected next action

## Acceptance Criteria

- AC1: Skill exists at `skills/status/SKILL.md` with proper YAML header
- AC2: Reports SDLC stage completion based on artifact presence (definition → architecture → planning → implementation → QA)
- AC3: Reports task counts by status directory
- AC4: Identifies the next expected SDLC action
- AC5: Flags at least: active tasks with no matching code changes, missing artifacts for completed stages, tasks in wrong directory for their YAML status
- AC6: Read-only — does not modify any files
- AC7: README.md and spec updated

## Quality Gates

- Verify against a project in mid-implementation: status report correctly identifies completed stages, active work, and next action
