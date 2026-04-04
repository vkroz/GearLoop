---
status: backlog
type: task
phase: 3
depends_on: []
---

# Add SDLC document review to codebase-review

## Problem

`codebase-review` checks code quality (dead code, DRY, naming, etc.) but does not check code against `docs/architecture.md` or `docs/spec.md`. Misses architectural drift and spec non-compliance — the problems that matter most in a document-driven SDLC.

## Scope

**In scope:**
- Add review category: "Architecture compliance" — code structure, patterns, and technology choices match `docs/architecture.md`
- Add review category: "Spec compliance" — implemented behavior matches `docs/spec.md` and `docs/requirements.md`
- These categories apply only when the corresponding docs exist (graceful skip if not present)

**Out of scope:**
- Changing existing review categories (1–10)
- Modifying the task creation workflow after review

## Acceptance Criteria

- AC1: Codebase review checklist includes "Architecture compliance" category with specific check items
- AC2: Codebase review checklist includes "Spec compliance" category with specific check items
- AC3: Both categories are skipped gracefully when docs are absent (no false errors)
- AC4: Findings from these categories follow the same severity format as existing categories

## Quality Gates

- Review checklist covers: technology choice alignment, component boundary compliance, API pattern adherence, data model consistency (architecture); feature completeness, behavioral correctness, edge case handling (spec)
