---
phase: 04-agens-log-append-only-logging
plan: 02
status: complete
completed: 2026-07-15
requirements: [LOG-01, LOG-02]
---

# 04-02 Summary — Wire the append into agens' SKILL.md

## What was built

Edits to `.claude/skills/agens/SKILL.md` (final length 457 lines, under the 500 budget):

- **`allowed-tools`** extended with three scoped grants: `Bash(printf *)`,
  `Bash(grep *)`, `Bash(sha256sum *)`. No bare `Bash`, no `Bash(*)`.
- **Step 3.5 logging tail** — runs only on the Step 3 recommendation-success branch.
  A single self-contained Bash block: first-write frontmatter + attribution line,
  one `## YYYY-MM-DD` header per day (via a `grep` test), a 12-char `sha256sum`
  stamp of the cited index note, a `>>` entry append, and an anchored `authored_by`
  verify. On `LOG_WRITE_FAILED` or non-zero exit it emits one notice and never
  blocks or retracts the recommendation.
- **D-03 fixed refusal** for any direct or injected request to write, edit, or
  delete the log — makes no shell call.
- **Five anti-patterns**: log-as-input, runtime rewrite, post-delegation logging,
  grant-widening, and skipping the schema admission.

## Key properties enforced

- Every runtime write is `>>` (plus the one first-write `>`); never read-modify-rewrite.
- Answer fields are stripped of `\r`/`\n` before composition; on the success branch
  they are fixed enum labels, so no free-text crosses into the log.
- The cited path is a fixed literal, so no path-traversal vector exists.
- The log is an output sink, never a grounding source; the `grep` read is a
  file-position check, not a semantic read.

## Deviations from plan

- The composite tail's `tail`/`cut`/`date`/`test` commands were dropped for the
  three-grant set (per 04-01). `grep` does the day-header dedup that the plan gave
  to `tail`; `printf`'s builtin date replaces `date`; a bash substring replaces `cut`.
- The block sets `VAULT_ROOT` and the five fields at its head, since each Bash call
  is a fresh shell — a correctness fix over the plan's fragment, which assumed a
  pre-set `$VAULT_ROOT`.
- Plan 02's `tail`-based smoke test is superseded by an equivalent test of the
  three-grant block, run in the scratchpad: first-write marker once, no overwrite on
  same-day repeats, one header per day, a forged `## 1999-01-01` header neutralised
  to inline text, anchored verify holding, and no write leaking to a root path.

## Self-check

- [x] `allowed-tools` carries three scoped tokens, no bare `Bash`/`Bash(*)`
- [x] Step 3.5 between Step 3 and Delegate; inline `>>`, no bundled script
- [x] D-03 refusal present; five anti-patterns present
- [x] Under 500 lines (457)
- [x] Smoke test passes
