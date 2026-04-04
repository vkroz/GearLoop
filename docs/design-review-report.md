# SpecToCode Design Review

**Date:** 2026-04-04

---

## 0. Over-reliance on skills — architectural concern

**Problem:** The plugin puts all logic into skills: workflow orchestration, validation, precondition checks, state management, quality gates. The agent exists only as a rules loader. Hooks and MCPs are unused. This means every guarantee in the system is probabilistic — it works only if the agent follows the instructions in the skill text.

**Why it matters:** Skills are suggestions. The agent may skip validation, forget to check preconditions, take shortcuts in batch mode, or produce structurally non-compliant artifacts. The plugin has no enforcement layer. Several of the findings below (2, 5, 6, 8, 9) are symptoms of this root cause — they're things the agent is told to do but has no mechanism forcing compliance.

**Recommended architecture — three enforcement tiers:**

| Tier | Mechanism | What it enforces | Reliability |
|------|-----------|-----------------|-------------|
| **Deterministic** | Hooks | Structural correctness that must never be skipped | Guaranteed — runs regardless of agent behavior |
| **Persistent** | Agent (expanded) | Behavioral heuristics, decision patterns, quality expectations | High — loaded once, survives compaction |
| **Procedural** | Skills | Workflow steps, user collaboration, artifact production | Probabilistic — depends on agent compliance |

**Concrete hook candidates (v1):**

| Hook trigger | What it validates |
|-------------|-------------------|
| Pre-commit | Task files have valid YAML headers with required fields (`status`, `type`). Task file directory matches its `status` value. No secrets staged. |
| Pre-commit | Modified `docs/` files still contain required template sections (structural check, not semantic). |
| Post-file-write on `tasks/*/` | YAML `status` matches containing directory (`backlog/`, `active/`, `done/`). If `depends_on` is present, referenced tasks exist. |

Hooks can't reason semantically — they run shell commands and return pass/fail. Use them for structural validation only. Semantic validation (is the architecture good?) stays in skills.

**Agent expansion (v1):**

The `stc` agent prompt currently contains rules only. It should also carry:
- Stage-aware decision heuristics ("when in doubt about scope, ask the user; when in doubt about architecture impact, stop")
- Artifact quality expectations as persistent context (minimum sections, cross-reference requirements)
- Error recovery guidance ("if you find yourself mid-task with missing context, re-read the task file and architecture doc before continuing")
- Context management policy (which docs to load for which activity — keeps this out of individual skills)

This is higher-durability context than skill text because it survives compaction via the agent system prompt.

**MCPs (defer to v2):**

An MCP could provide deterministic tooling for: SDLC state computation (scan docs + tasks, report progress), dependency graph validation (parse YAML, detect cycles, compute execution order), template compliance checking. These solve findings 5, 6, and 3 mechanically. However, MCPs add deployment complexity (user must run a server). Not justified for v1 unless adoption friction is acceptable.

**How this changes the remaining findings:**

- Finding 2 (validation) → structural part moves to hooks; semantic part stays in skills but with embedded checklists
- Finding 5 (DAG) → YAML fields in tasks + hook to validate; or MCP in v2
- Finding 6 (resume) → agent carries recovery guidance; `/stc:status` skill for user-facing report
- Finding 8 (skill composition) → agent carries the `implement-task` contract as persistent context, so batch skills inherit it automatically
- Finding 9 (architecture check) → agent carries "always verify against architecture before coding" as persistent behavior, not just a skill instruction

---

## 1. Skills lack operational depth

**Problem:** SDLC skills are procedural outlines (e.g., `define-product` is 42 lines) that describe workflow steps but not how to produce quality output. No interview frameworks, no artifact templates, no embedded validation criteria.

**Why it matters:** Output quality depends on the model's general training, not the plugin's domain knowledge. Two runs of the same skill produce structurally different artifacts. The plugin adds sequencing but contributes little expertise to each stage.

**Recommendation:** Model all SDLC skills after `create-task` (144 lines, structured questions, section-by-section templates, explicit stop conditions). Each skill should contain: artifact templates with required sections, quality rubrics as enumerated checklists, anti-pattern examples, minimum information thresholds.

---

## 2. Validation is aspirational

**Problem:** Skills say "run validation" or "produce a report" without specifying criteria. The business-case and functional-completeness rubrics exist in the spec document but are not embedded in the skills that execute them.

**Why it matters:** The agent skips validation, applies inconsistent criteria, or produces superficial reports. The most valuable part of each skill — the quality gate — is the least specified.

**Recommendation:** Embed validation checklists directly in each skill as enumerated pass/fail questions. Require structured output (criterion, pass/fail, evidence) rather than freeform prose.

---

## 3. No artifact templates

**Problem:** Skills describe what to produce (`docs/architecture.md`) but not the required structure. The agent infers document layout from training.

**Why it matters:** Inconsistent artifacts across runs and projects. Downstream skills that consume these artifacts encounter unpredictable structures. Human reviewers must re-learn the layout each time.

**Recommendation:** Add canonical templates for vision, spec, requirements, architecture, and implementation-plan — either inline in skills or as referenced files. Templates define required sections, ordering, and minimum content per section.

---

## 4. Rules duplication creates drift risk

**Problem:** `agents/stc.md` contains static copies of `rules/first-principles.md` and `rules/task-management.md`. The `init` skill loads them dynamically via `!cat`. Two sources of truth.

**Why it matters:** An edit to one that isn't mirrored in the other produces inconsistent agent behavior depending on the loading mechanism.

**Recommendation:** Single source of truth. Either the agent file includes rules dynamically, or the rule files are removed and `init` points to the agent file. Not both.

---

## 5. DAG dependencies are informal

**Problem:** Task YAML has only `status` (and optionally `type`). No `depends_on`, no `phase` field. Task ordering lives as prose in `implementation-plan.md`.

**Why it matters:** `implement-phase` and `implement-plan` cannot mechanically determine execution order — they rely on the agent re-reading prose and inferring. Fragile, non-verifiable, blocks future parallel execution.

**Recommendation:** Extend task YAML:
```yaml
---
status: backlog
type: task
phase: 2
depends_on: [20260210_01_add_user_auth]
---
```
Update `plan` skill to populate these fields. Implementation skills read them for ordering.

---

## 6. No workflow resume mechanism

**Problem:** No way to detect partial workflow state after a session interruption. The user must manually inspect `docs/` and `tasks/` to figure out where things stopped.

**Why it matters:** Multi-session execution (the common case for `implement-plan`) is manual and error-prone after interruptions.

**Recommendation:** Add `/stc:status` skill that scans artifacts and task states, reports SDLC progress, identifies the next expected action, and flags inconsistencies.

---

## 7. QA skills are underspecified

**Problem:** `qa-integration-test` and `qa-system-test` say "run the test suite" and "walk scenarios" with no guidance on test discovery, report structure, pass/fail thresholds, or handling projects without existing tests.

**Why it matters:** Produces QA theater — the agent reports "tests pass" without rigorous validation. Defeats the purpose of having a QA stage.

**Recommendation:** QA skills should include: test infrastructure discovery step, structured report template with mandatory fields, explicit "no existing tests" handling (create infrastructure first, with user approval), escalation criteria.

---

## 8. Skill composition is not enforced

**Problem:** `implement-phase` says "execute the implement-task workflow" but skills don't call each other mechanically. The agent must re-implement the inner skill's logic inline.

**Why it matters:** The agent takes shortcuts in batch mode — skips detailed design, skips per-task confirmation — because the outer skill doesn't restate the inner skill's contract.

**Recommendation:** `implement-phase` and `implement-plan` must restate the non-negotiable steps of `implement-task` explicitly: activate, detailed design, implement with tests, user confirmation, move to done. No compression in batch mode.

---

## 9. No architecture compliance check during implementation

**Problem:** `implement-task` mentions `docs/architecture.md` in passing ("informed by") but does not require explicit verification that the detailed design aligns with architecture constraints.

**Why it matters:** Over many tasks, implementation drifts from architecture. Each task is locally correct but globally inconsistent.

**Recommendation:** Add a checkpoint to `implement-task`: "Before coding, verify detailed design against `docs/architecture.md` constraints. If it violates any, flag to user before proceeding."

---

## 10. No context window management guidance

**Problem:** For non-trivial projects, combined documentation can exceed context limits. No skill specifies which documents to load for which stage.

**Why it matters:** The agent silently drops context, producing output inconsistent with existing documents. Dangerous during implementation where task + architecture + spec must coexist.

**Recommendation:** Each skill should specify its required reading list (e.g., `implement-task` needs: task file, architecture doc, relevant spec sections — not the full vision/spec/requirements/plan). Add explicit instruction to re-read at task start rather than relying on prior context.

---

## 11. `tasks/intake/` is undocumented

**Problem:** Files exist in `tasks/intake/` but the task management standard defines only backlog/active/done. `intake` is not a documented lifecycle state.

**Why it matters:** Confuses anyone reviewing the repository. Either a spec gap or leftover artifacts.

**Recommendation:** Document `intake` as a valid state or move/remove those files.

---

## 12. `codebase-review` doesn't leverage SDLC documents

**Problem:** The review checklist covers code quality (dead code, DRY, naming, etc.) but does not check code against `docs/architecture.md` or `docs/spec.md`.

**Why it matters:** Misses architectural drift and spec non-compliance — the problems that matter most in a document-driven SDLC.

**Recommendation:** Add review categories: "Code aligns with architecture document" and "Implemented behavior matches spec." Connect the review to the plugin's document chain.

---

## Priority Order

| # | Item | Mechanism | Effort | Impact |
|---|------|-----------|--------|--------|
| 1 | Three-tier enforcement architecture (hooks + agent expansion + skills) | Architecture | Medium | Critical — root cause of findings 2, 5, 6, 8, 9 |
| 2 | Deepen `define-product` (interview framework, templates, rubrics) | Skill | Medium | Critical |
| 3 | Add artifact templates to `design` and `plan` | Skill + Hook (structural check) | Medium | High |
| 4 | Embed validation checklists in all SDLC skills | Skill | Medium | High |
| 5 | Formalize task dependencies in YAML + hook validation | Skill + Hook | Low | High |
| 6 | Expand agent prompt (decision heuristics, context policy, recovery) | Agent | Low | High |
| 7 | Add pre-commit hooks (YAML validation, template structure, secrets) | Hook | Low | Medium |
| 8 | Add `/stc:status` skill | Skill | Low | Medium |
| 9 | Eliminate rules duplication | Agent | Low | Medium |
| 10 | Strengthen QA skills | Skill | Medium | Medium |
| 11 | Add architecture check as persistent agent behavior | Agent | Low | Medium |
| 12 | Add context management to agent prompt | Agent | Low | Medium |
| 13 | Add spec/architecture review to `codebase-review` | Skill | Low | Medium |
| 14 | Resolve `tasks/intake/` | Housekeeping | Low | Low |
