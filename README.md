# spec-to-code

Claude Code plugin for AI-assisted SDLC: definition, architecture, planning, implementation, QA.


## Install


## Getting started

1. Install the plugin. 

```bash
claude plugin install spec-to-code@<marketplace-name>
```

or install from local directory:
```bash
claude --plugin-dir /path/to/spec-to-code
```

2. The `stc` agent activates automatically and loads behavioral rules. If your setup cannot use the `stc` agent, run `/stc:load-rules` at session start instead.

3. Use plugin skills to run the workflows. Run `/stc:define-product` to start a new product, or `/stc:create-task` for a one-off task or bug ticket. See "Skills" section below for more details.

## Skills

### SDLC workflow (run in order for new products)

| Step | Command | What it does |
|------|---------|--------------|
| 0. Init (if no agent) | `/stc:load-rules` | Load rules into session context |
| 1. Definition | `/stc:define-product` | Vision, spec, requirements in `docs/` |
| 2. Architecture | `/stc:design` | `docs/architecture.md` |
| 3. Planning | `/stc:plan` | `docs/implementation-plan.md` + task files in `tasks/backlog/` |
| 4. Implementation | `/stc:implement task <path>` | One task: detailed design, code, tests |
| | `/stc:implement plan <path>` | Full plan, phase by phase |
| | `/stc:implement plan <path> phase N` | Specific phase(s) of a plan |
| 5. Integration test | `/stc:qa-integration-test` | Verify phase or ad-hoc work + regression |
| 6. System test | `/stc:qa-system-test` | Full product test, release gate |

### Task and maintenance

| Command | What it does |
|---------|--------------|
| `/stc:create-task` | New task or bug ticket in `tasks/backlog/` (dev tasks, bugs, refactors, chores) |
| `/stc:codebase-review` | Read-only audit, produces backlog tasks |

### Utilities

| Command | What it does |
|---------|--------------|
| `/stc:commit-code` | Staging and git commit guidance |
| `/stc:comment-code` | Add explanatory comments to code |

## Workflows supported

- **New product** — `/stc:define-product` → `/stc:design` → `/stc:plan` → `/stc:implement plan` → `/stc:qa-system-test`
- **Bug fix** — `/stc:create-task bug` → `/stc:implement task` → `/stc:qa-integration-test` → `/stc:qa-system-test`
- **Maintenance** — `/stc:create-task task` → `/stc:implement task` → `/stc:qa-integration-test`
- **Incremental feature** — same as new product, but `/stc:define-product` amends existing docs
- **Codebase optimization** — `/stc:codebase-review` → `/stc:plan` → `/stc:implement plan ... phase N` → `/stc:qa-system-test`

## Example use cases

- **Build a new product from scratch** — `/stc:define-product` → `/stc:design` → `/stc:plan` → `/stc:implement plan` → `/stc:qa-system-test`. Claude facilitates writing specs, designs the architecture, generates a phased task plan, then autonomously implements and tests each task.
- **Fix a bug** — `/stc:create-task bug` → `/stc:implement task` → `/stc:qa-integration-test`. Claude captures the defect, investigates root cause, implements the fix with tests, and validates no regressions.
- **Add a feature to an existing product** — `/stc:define-product` amends existing docs with the new feature, then the standard design → plan → implement → QA flow executes scoped to that feature.
- **Audit and reduce technical debt** — `/stc:codebase-review` produces a prioritized report of duplication, drift, and fragmentation; `/stc:plan` → `/stc:implement plan ... phase N` executes the approved improvements.

## Project layout created by workflows

```
docs/vision.md
docs/spec.md
docs/requirements.md
docs/architecture.md
docs/implementation-plan.md
tasks/backlog/ | active/ | done/
```

Created incrementally as you run each stage — no upfront scaffolding.

## How rules are loaded

| Mechanism | How | Survives compaction | When to use |
|-----------|-----|---------------------|-------------|
| `stc` agent (default) | Automatic via `settings.json` | Yes | Recommended for most users |
| `/stc:load-rules` | Manual, run once per session | No | When agent conflicts with another plugin |

Rules live in `rules/first-principles.md` and `rules/task-management.md`. The agent inlines them in its system prompt; `/stc:load-rules` injects them via `!cat`.

## Development and testing

```bash
# Load the plugin locally without installing:
claude --plugin-dir ./

# After making changes, reload inside Claude Code:
/reload-plugins
```

See [`docs/spec-to-code-specification.md`](docs/spec-to-code-specification.md) for the full product vision and functional specification.
