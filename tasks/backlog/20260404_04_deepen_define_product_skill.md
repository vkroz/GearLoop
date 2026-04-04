---
status: backlog
type: task
phase: 2
depends_on: [20260404_01_consolidate_agent_prompt]
---

# Deepen define-product skill

## Problem

`define-product` is a 42-line procedural outline. It says "Interview the user" without providing an interview structure. It says "Validate" without embedding the validation criteria. It produces no artifact templates. Compare this to `create-task` (144 lines, structured questions, section templates, explicit stop conditions) — `define-product` drives the highest-leverage SDLC stage with the least guidance.

## Scope

**In scope:**
- Add structured interview framework for each artifact (vision, spec, requirements) — required questions, depth guidance, what "done" looks like for each
- Add artifact templates with required sections for `docs/vision.md`, `docs/spec.md`, `docs/requirements.md`
- Embed business-case and functional-completeness validation rubrics as enumerated pass/fail checklists (currently described only in the spec document)
- Add minimum information thresholds (e.g., "spec must address at minimum: user roles, core workflows, data model scope, integration points")
- Add anti-pattern examples (what a bad vision/spec/requirements document looks like)

**Out of scope:**
- Changing the workflow steps themselves (interview → draft → iterate → validate → gate)
- Adding new artifacts beyond vision, spec, requirements
- Technology-stack-specific guidance

## Acceptance Criteria

- AC1: Skill contains interview framework for each of the three artifacts with required questions
- AC2: Skill contains artifact templates for vision, spec, and requirements with required sections listed
- AC3: Business-case validation rubric is embedded as enumerated checklist (not just "run business case validation")
- AC4: Functional-completeness rubric is embedded as enumerated checklist
- AC5: Validation output format is specified: structured (criterion, pass/fail, evidence), not freeform
- AC6: Skill includes at least one anti-pattern example per artifact type
- AC7: Skill stays under 200 lines

## Quality Gates

- Dry-run: read the skill as an agent would and verify that every step has enough specificity to execute without guesswork
- Cross-check: every rubric item in `docs/spec-to-code-specification.md` (Definition stage) is represented in the skill's validation checklist
