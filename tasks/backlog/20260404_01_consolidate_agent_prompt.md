---
status: backlog
type: task
phase: 1
depends_on: []
---

# Consolidate agent: single source of truth + expanded behavioral context

## Problem

`agents/stc.md` contains static copies of `rules/first-principles.md` and `rules/task-management.md`. The `init` skill loads them dynamically via `!cat`. Two sources of truth that can drift. Additionally, the agent prompt carries only rules — it does not carry decision heuristics, context management policy, or error recovery guidance that should persist across all skills and survive compaction.

## Scope

**In scope:**
- Eliminate rules duplication: one canonical location for all rule content
- Expand agent system prompt with: decision heuristics, artifact quality expectations, context management policy (which docs to load per activity), error recovery guidance, implementation contract (per-task steps that batch skills must not compress), architecture compliance as persistent behavior
- Update `init` skill to remain consistent with the chosen single-source approach

**Out of scope:**
- Changing rule content itself (YAGNI, KISS, etc.)
- Modifying skill workflow logic
- Adding new skills

## Acceptance Criteria

- AC1: All rule content exists in exactly one location. No text is duplicated between `agents/stc.md` and `rules/*.md`.
- AC2: Agent prompt contains decision heuristics section (scope doubt → ask user, architecture doubt → stop, missing context → re-read docs)
- AC3: Agent prompt contains context management policy (required reading list per SDLC activity)
- AC4: Agent prompt contains the per-task implementation contract (activate → detailed design → implement with tests → user confirmation → done) stated as persistent behavioral requirement
- AC5: Agent prompt contains architecture compliance directive ("verify detailed design against architecture doc before coding")
- AC6: `/stc:init` loads the same content as the agent mechanism — no behavioral difference between the two loading paths
- AC7: Agent prompt stays under 3000 words to leave context room for skills and user work

## Quality Gates

- Run `/stc:init` and compare loaded context against agent system prompt — must be equivalent
- Review agent prompt for internal consistency (no contradictions between heuristics and rules)
- Verify no orphaned rule files remain that could mislead contributors
