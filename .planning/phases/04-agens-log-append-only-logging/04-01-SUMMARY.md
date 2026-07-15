---
phase: 04-agens-log-append-only-logging
plan: 01
status: complete
completed: 2026-07-15
requirements: [LOG-01, LOG-02]
---

# 04-01 Summary — Write mechanism decision and schema admission

## What was decided

**Mechanism: PRIMARY inline `>>`, three declared grants.** The Wave 0 probe was
built (`~/.claude/skills/agens-grant-probe`) but not run as designed. On review,
the user judged the probe's Form 1 (chaining ungranted commands behind one grant)
and Form 3 (`${CLAUDE_SKILL_DIR}` script token) to be a test of permission-boundary
evasion, and directed the Form 2 answer instead: declare every command explicitly.

The composite tail was simplified to a plain timestamped append needing three
commands, each named in `allowed-tools`:

- `Bash(printf *)` — writes the entry and day header (`%(...)T` supplies the date, so no `date` grant)
- `Bash(grep *)` — verifies the `authored_by` marker and tests today's header (replaces the composite tail's `tail` read)
- `Bash(sha256sum *)` — stamps the cited note's bytes (12-char via bash substring, so no `cut` grant)

`${CLAUDE_SKILL_DIR}`-in-`allowed-tools` was not tested (fallback path not taken).
The prompt-free question was answered later by live use in Plan 03: the block runs
with no permission prompt.

## What was built

- **`wiki-agents/99_Meta/schema.md` §2.8** — admits `authored_by` with the single
  controlled value `agens`, in the `Field | Enum values | Notes` shape. The meta
  note's own frontmatter is unchanged (it stays human-authored).

## Deviations

- Probe not executed; mechanism set by user decision, not empirical probe.
- Grant set is three declared tokens, not the composite tail's 6-7 commands.
- The probe skill was created then deleted (delete completed after the user closed
  the Claude instance that held the folder open as its working directory).

## Self-check

- [x] Mechanism decided before SKILL.md wiring
- [x] Schema admits `authored_by` before first write
- [x] No script authored on the primary path
- [x] Probe skill removed; nothing committed to `.claude/skills/agens/` from the probe
