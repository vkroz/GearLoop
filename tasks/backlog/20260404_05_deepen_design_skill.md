---
status: backlog
type: task
phase: 2
depends_on: [20260404_01_consolidate_agent_prompt]
---

# Deepen design skill

## Problem

`design` skill says "Draft `docs/architecture.md`" with no template, no required sections, no evaluation criteria. The agent infers architecture document structure from training. Produces inconsistent architecture documents across projects. Downstream skills (plan, implement-task) encounter unpredictable structures when reading the architecture.

## Scope

**In scope:**
- Add artifact template for `docs/architecture.md` with required sections and guidance per section
- Define minimum information thresholds (e.g., must address: deployment topology, data flow, external integrations, security boundaries, technology choices with rationale)
- Embed architecture validation rubric as enumerated checklist: completeness against spec, over/under-engineering, design principle compliance, consistency with definition docs
- Specify validation output format (structured, not freeform)

**Out of scope:**
- Technology-stack-specific architecture patterns
- Detailed design templates (those belong in implement-task)
- Changing the workflow (draft → review → validate → consistency check → gate)

## Acceptance Criteria

- AC1: Skill contains architecture document template with required sections
- AC2: Minimum information thresholds are listed
- AC3: Validation rubric is embedded as enumerated pass/fail checklist
- AC4: Validation output format is specified as structured
- AC5: Skill stays under 150 lines

## Quality Gates

- Template sections cover all areas mentioned in the spec's Architecture stage description
- Validation checklist covers all review criteria from `docs/spec-to-code-specification.md`
