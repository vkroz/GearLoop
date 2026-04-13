---
status: backlog
type: bug
---

# /stc:load-rules skill blocked by Claude Code sandbox when invoked from external projects

**Date:** 2026-04-04

## Observed problem

Running `/stc:load-rules` (previously `/stc:init` — renamed 2026-04-07 to avoid clash with Claude Code's built-in `/init`; see `tasks/done/20260407_01_init_skill_name_clash.md`) from a project directory outside the SpecToCode repository fails with a shell command permission error. The skill cannot load its rule files because Claude Code's security sandbox blocks `cat` from reading files outside the session's allowed working directories.

This breaks the primary use case of the plugin — being installed into and invoked from external projects.

## Expected vs actual

**Expected:** `/stc:load-rules` loads `first-principles.md` and `task-management.md` into the session context regardless of which project directory the user is working in.

**Actual:** Claude Code blocks the shell commands with:

```
Error: Shell command permission check failed for pattern
"!cat /Users/.../SpecToCode/skills/load-rules/../../rules/first-principles.md":
cat in '/Users/.../SpecToCode/rules/first-principles.md' was blocked.
For security, Claude Code may only concatenate files from the allowed
working directories for this session: '/Users/.../numio-app'.
```

The same error occurs for `task-management.md`.

## Reproduction steps

1. Install SpecToCode as a Claude Code plugin in any project outside the SpecToCode repo (e.g., `~/repos/numio-root/numio-app`).
2. Open a Claude Code session in that external project.
3. Run `/stc:load-rules`.
4. Observe the permission error for both rule files.

## Environment & context

- **Affected file:** `skills/load-rules/SKILL.md` (lines 14 and 18)
- **Root cause:** The skill uses `!cat ${CLAUDE_SKILL_DIR}/../../rules/first-principles.md` and `!cat ${CLAUDE_SKILL_DIR}/../../rules/task-management.md`. `${CLAUDE_SKILL_DIR}` resolves to the skill's location inside the SpecToCode repo, which is outside the session's sandbox when invoked from an external project.
- **Scope:** Only the `load-rules` skill uses this pattern. No other skills reference rule files via `cat` with relative paths.
- **Downstream impact:** Since `/stc:load-rules` loads the rules that govern all `/stc:*` skills, this bug prevents the entire plugin from functioning correctly in external projects. Skills that use the `stc` agent (which loads rules via its agent definition) are unaffected, but standalone `/stc:load-rules` invocation is broken.

## Risks & impact

- **Severity:** High — blocks plugin usage from external projects, which is the primary deployment model.
- **Blast radius:** All users who install SpecToCode as a plugin and invoke `/stc:load-rules` from their own project directories.
- **Workaround:** Users can use `/stc:*` skills directly if the `stc` agent is configured, since the agent definition loads rules through a different mechanism. However, `/stc:load-rules` itself remains broken for standalone use.
