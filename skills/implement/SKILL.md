---
name: implement
description: Execute implementation work — a single task or a plan (all phases or specific ones). Invoke with /stc:implement task <path> or /stc:implement plan <path> [phase N]. If mode is omitted, ask the user.
argument-hint: task <path> | plan <path> [phase N,M,...]
---

# Implement (`/stc:implement`)

## Arguments

| Position | Variable | Value |
|----------|----------|-------|
| `$0` | mode | `task` or `plan` |
| `$1` | path | Task file path (mode `task`) or plan file path (mode `plan`) |
| `$2` | *(optional)* | Literal `phase` keyword (plan mode only) |
| `$3` | *(optional)* | Phase number(s), comma-separated (plan mode only) |

## Usage

```
# Implement a single task
/stc:implement task @tasks/backlog/20260210_add_feature.md

# Implement an entire plan (all phases)
/stc:implement plan @docs/implementation-plan.md

# Implement specific phase(s) of a plan
/stc:implement plan @docs/implementation-plan.md phase 1
/stc:implement plan @docs/implementation-plan.md phase 2,3,4
```

## Argument validation

If `$0` is missing or is not `task` / `plan`, **stop and ask**:

> What would you like to implement?
> - **task** `<path>` — a single task file
> - **plan** `<path>` — all phases of an implementation plan
> - **plan** `<path>` **phase** `<N,M,...>` — specific phase(s) of a plan

Do not guess; wait for the user's answer.

If `$0` is present but `$1` (path) is missing, ask the user for the file path.

---

## Mode: `task`

Activated when `$0` = `task`. Operates on the task file at `$1`.

### Preconditions
- A valid task file under `tasks/backlog/` or `tasks/active/` (or path given by user).
- For work that **changes high-level architecture**, stop and discuss with the user before coding (**architecture impact gate**).

### Required reading (re-read at task start, do not rely on prior context)
- The task file itself (`$1`)
- `docs/architecture.md`
- Relevant sections of `docs/spec.md` and `docs/requirements.md` referenced by the task

### Execution steps

1. **Activate the task**
   - Move the task file to `tasks/active/` if it is in `tasks/backlog/`.
   - Set `status: active` in the task file YAML header.

2. **Read and understand the task**
   - Read all items from the required reading list above.
   - **Detailed design first** — API shapes, data touched, interfaces, key logic — before writing any code.

3. **Architecture compliance checkpoint**
   - Verify the detailed design against `docs/architecture.md` constraints.
   - If it violates any constraint, **flag to the user before coding** — do not proceed until resolved.

4. **Execute the task**
   - Ask for clarification if the task is ambiguous.
   - For multi-step or risky work, get user approval before executing.
   - Implement, then write unit tests as appropriate.
   - Stay within task scope; flag scope creep to the user.

5. **Complete the task (after user confirms)**
   - Set `status: done` in the task file YAML header.
   - Move the task file to `tasks/done/`.

6. **Documentation** — If behavior or intent in docs is wrong or incomplete, update the relevant `docs/*` files and say what changed (dependency order: vision → spec → requirements → architecture → implementation plan → tasks).

### After single-task flows (bugfix / ad-hoc)
When the user expects integration coverage, run **`/stc:qa-integration-test`** for the affected scope before hand-off. Full regression uses **`/stc:qa-system-test`**.

---

## Mode: `plan`

Activated when `$0` = `plan`. Reads the plan file at `$1`.

- If `$2` = `phase` and `$3` is present → execute only the specified phase(s).
- If `$2` is absent → execute **all remaining phases** in order.

### Preconditions
- The plan file at `$1` exists and lists phases and tasks.
- Task files exist under `tasks/backlog/` (or paths referenced by the plan).

If the plan file does not exist: *Cannot run — complete `/stc:plan` first.*

### Workflow

1. **Read the plan** at `$1` and enumerate phases in order.
   - If specific phases were requested (`$3`), filter to those phases only.
   - Parse comma-separated values (e.g., `2,3,4`) to select multiple phases.

2. For **each phase** (in order), identify its tasks in DAG order (respect dependencies).

3. For **each task** in the phase, execute the full per-task contract — these steps **cannot be compressed or skipped in batch mode**:
   - **Activate** — move to `tasks/active/`, set `status: active`.
   - **Required reading** — re-read the task file, `docs/architecture.md`, and relevant spec sections at task start.
   - **Detailed design** — API shapes, data touched, interfaces, key logic.
   - **Architecture compliance checkpoint** — verify design against `docs/architecture.md`; flag violations before coding.
   - **Implement with tests** — code + unit tests; stay within scope.
   - **User confirmation** — wait for user to confirm the task is complete.
   - **Complete** — set `status: done`, move to `tasks/done/`.

   **v1:** run tasks **sequentially** (no parallel agent execution).

4. If a task is blocked, stop and report; do not skip silently.

5. **Phase boundary** — After all tasks in a phase are `done`, run **`/stc:qa-integration-test`** with scope = this phase + regression against prior phases.

6. **Gate** — User reviews integration results before starting the next phase.

7. **Interrupts** — The user may stop after any phase; report what is complete and what remains.

8. When all requested phases are `done`, point the user to **`/stc:qa-system-test`** for full product verification.

### Discipline
- Respect scope in the plan; flag conflicts with the user.
- Do not start the next phase without user approval.
- Do not expand scope beyond the approved plan without user instruction.
- Same architecture-impact rules as mode `task`.
