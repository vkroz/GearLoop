---
status: backlog
type: task
phase: 3
depends_on: [20260404_01_consolidate_agent_prompt]
---

# Strengthen implementation skills

## Problem

`implement-phase` and `implement-plan` say "execute the implement-task workflow" without restating its non-negotiable steps. The agent takes shortcuts in batch mode — skips detailed design, skips per-task confirmation. Additionally, `implement-task` mentions `docs/architecture.md` in passing but does not require explicit architecture compliance verification before coding.

## Scope

**In scope:**
- `implement-task`: add explicit architecture compliance checkpoint before coding step. Add required reading list (task file, architecture doc, relevant spec sections).
- `implement-phase`: restate the per-task contract inline (activate → detailed design → architecture check → implement with tests → user confirmation → done). Make clear these steps cannot be compressed.
- `implement-plan`: same as implement-phase — restate the contract, add phase-boundary requirements.
- Add context management guidance: which docs to load, instruction to re-read at task start.

**Out of scope:**
- Changing the fundamental workflow (steps stay the same, just better specified)
- Adding new implementation steps
- Parallel execution (deferred)

## Acceptance Criteria

- AC1: `implement-task` contains an explicit step: "Verify detailed design against `docs/architecture.md`. If it violates constraints, flag to user before coding."
- AC2: `implement-task` specifies required reading list
- AC3: `implement-phase` restates all per-task steps inline (not by reference to implement-task)
- AC4: `implement-plan` restates all per-task steps and phase-boundary requirements inline
- AC5: All three skills include context management instruction (re-read relevant docs at task start)

## Quality Gates

- Read `implement-phase` in isolation — it must be possible to execute the per-task contract without referencing `implement-task`
- Architecture check step has clear go/no-go criteria
