---
status: backlog
type: task
phase: 2
depends_on: [20260404_01_consolidate_agent_prompt]
---

# Strengthen QA skills

## Problem

`qa-integration-test` and `qa-system-test` say "run the test suite" and "walk scenarios" with no guidance on test discovery, report structure, pass/fail thresholds, or handling projects without existing tests. Produces QA theater — the agent reports "tests pass" without rigorous validation.

## Scope

**In scope:**
- Add test infrastructure discovery step to both skills: detect test framework, runner, test directory, existing coverage
- Add structured report template with mandatory fields (scope, what was executed, pass/fail per scenario, regressions, coverage delta, open issues, sign-off recommendation)
- Add explicit handling for "no existing tests" case: create test infrastructure first with user approval, define minimum coverage expectations
- Add escalation criteria: when to stop and ask the user vs. when to fix autonomously
- Differentiate the two skills clearly: integration = scope-bounded verification; system = full-product verification with release gate

**Out of scope:**
- Prescribing specific test frameworks
- Writing test code (that's the implementation skill's job)
- Automated test execution tooling

## Acceptance Criteria

- AC1: Both skills contain a test discovery step
- AC2: Both skills contain a structured report template with mandatory fields
- AC3: Both skills handle the "no existing tests" case explicitly
- AC4: Escalation criteria are defined (severity thresholds, decision types that require user)
- AC5: Clear differentiation between integration and system test scope
- AC6: Each skill stays under 100 lines

## Quality Gates

- Report template fields cover all outputs described in the spec for each QA stage
- "No tests" handling does not skip QA — it establishes a minimum baseline
