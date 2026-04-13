---
status: backlog
type: task
phase: 3
depends_on: [20260404_01_consolidate_agent_prompt]
---

# Strengthen implementation skill

## Problem

The three separate implementation skills (`implement-task`, `implement-phase`, `implement-plan`) have been consolidated into a single `implement` skill with modes: `task`, `phase`, `all`. The consolidated skill already restates the full per-task contract in batch modes and includes an architecture compliance checkpoint. Remaining work is to verify the unified skill meets all original acceptance criteria and fine-tune if gaps remain.

## Scope

**In scope:**
- Verify unified `implement` skill restates per-task contract inline in `phase` and `all` modes (no compression)
- Verify architecture compliance checkpoint is present and has clear go/no-go criteria
- Verify required reading list is specified for task mode
- Verify context management guidance (re-read at task start) is present in all modes
- Fine-tune wording if any AC is not fully met

**Out of scope:**
- Restructuring the skill (consolidation is already done)
- Adding new implementation steps
- Parallel execution (deferred)

## Acceptance Criteria

- AC1: `implement` (task mode) contains explicit step: "Verify detailed design against `docs/architecture.md`. If it violates constraints, flag to user before coding."
- AC2: `implement` (task mode) specifies required reading list
- AC3: `implement` (phase mode) restates all per-task steps inline (not by reference)
- AC4: `implement` (all mode) restates all per-task steps and phase-boundary requirements inline
- AC5: All three modes include context management instruction (re-read relevant docs at task start)

## Quality Gates

- Read the `implement` skill in isolation — each mode must be self-contained and executable without cross-referencing other modes
- Architecture check step has clear go/no-go criteria
