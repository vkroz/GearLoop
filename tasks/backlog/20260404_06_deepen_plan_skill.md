---
status: backlog
type: task
phase: 2
depends_on: [20260404_01_consolidate_agent_prompt, 20260404_02_extend_task_yaml_schema]
---

# Deepen plan skill

## Problem

`plan` skill says "Draft implementation plan: phases, tasks, dependencies" with no guidance on task granularity, phase sizing, dependency modeling, or plan document structure. Generated tasks use only basic YAML fields (`status`), missing the `phase` and `depends_on` fields needed for mechanical execution ordering.

## Scope

**In scope:**
- Add artifact template for `docs/implementation-plan.md` with required sections
- Add guidance on task granularity (what makes a task too large or too small)
- Add guidance on phase sizing and dependency identification
- Update task generation to populate extended YAML fields (`phase`, `depends_on`) per the updated task management standard
- Embed planning validation rubric: task ambiguity check, spec coverage cross-check, dependency completeness, phase boundary rationale
- Specify validation output format

**Out of scope:**
- DAG visualization or tooling
- Dependency cycle detection (deferred to hooks/MCP)
- Changing task file body structure (that's defined in the task management standard)

## Acceptance Criteria

- AC1: Skill contains implementation plan template with required sections
- AC2: Task granularity and phase sizing guidance is included
- AC3: Skill instructs generation of `phase` and `depends_on` in task YAML headers
- AC4: Validation rubric is embedded as enumerated checklist
- AC5: Validation includes explicit spec coverage cross-check: "every requirement in spec/requirements is addressed by at least one task"
- AC6: Skill stays under 150 lines

## Quality Gates

- Generated task files from this skill must have valid extended YAML headers per task management standard
- Plan template covers all aspects described in the spec's Planning stage
