# Phase 4: agens-log (Append-Only Logging) - Discussion Log

> **Audit trail only.** Do not use as input to planning, research, or execution agents.
> Decisions are captured in CONTEXT.md — this log preserves the alternatives considered.

**Date:** 2026-07-15
**Phase:** 4-agens-log (Append-Only Logging)
**Areas discussed:** Packaging & LOG-02 enforcement

---

## Gray Area Selection

Four gray areas were presented; the user selected one to discuss.

| Area | Selected |
|------|----------|
| Packaging & LOG-02 enforcement | ✓ |
| Entry content & vault-state stamp | |
| Append mechanism & tool grant (header ordering details) | |
| Write failure & bounded read-back | |

The three unselected areas are recorded in CONTEXT.md as undecided, not defaulted.

---

## Packaging & LOG-02 enforcement

### Question 1 — How should the logging capability be packaged?

| Option | Description | Selected |
|--------|-------------|----------|
| Inline step in agens SKILL.md | No separate skill exists at all; LOG-02 holds structurally — no command, no auto-trigger surface, nothing to invoke. Trade-off: agens' allowed-tools gains a write mechanism; SKILL.md grows toward the 500-line budget. | ✓ |
| Separate agens-log skill, user-invocable: false | Sibling skill directory hidden from the / menu; agens calls it via the Skill tool. Isolates the write grant, but the Skill grant cannot be scoped, so enforcement rests on frontmatter plus prompt text. | |
| Inline step + lazy-loaded reference file | Invocation inline; detailed procedure in references/logging.md loaded only when the step runs. Keeps SKILL.md lean; still widens agens' own tool grant. | |

**User's choice:** Inline step in agens SKILL.md (the recommended option).
**Notes:** None.

### Question 2 — Which write mechanism should the inline logging step add to agens' allowed-tools?

| Option | Description | Selected |
|--------|-------------|----------|
| Scoped Bash append | Bash granted narrowly for the `>>` append. True append-only semantics; file never read-modify-rewritten. Matches CLAUDE.md's own ruling. | ✓ |
| Read-then-Write full rewrite | Write tool; whole-file rewrite per entry. The pattern CLAUDE.md rejects; one malformed rewrite can destroy prior entries. | |
| Edit-tool tail append | Edit tool anchored on the file tail. Brittle anchor; still requires a read the bounded-read-back criterion must police. | |

**User's choice:** Scoped Bash append (the recommended option).
**Notes:** None.

### Question 3 — When someone directly asks agens to write or edit the log outside a recommendation flow, what should happen?

| Option | Description | Selected |
|--------|-------------|----------|
| Fixed plain refusal | Scripted refusal line in SKILL.md, matching the Phase 1/2 house style; gives verification an exact assertion target. | ✓ |
| Body rule only, no scripted message | Internal rule; agens declines in its own words. Untestable; phrasing drifts. | |
| Allow read, refuse write | Viewing honoured, mutation refused with the fixed text. | |

**User's choice:** Fixed plain refusal (the recommended option).
**Notes:** None.

### Question 4 — How should Phase 4 handle the project-copy vs user-level deployed copy of agens?

| Option | Description | Selected |
|--------|-------------|----------|
| Edit project copy, redeploy to user level | Repo copy is the source of truth; a phase task copies to ~/.claude/skills/agens/ and verification checks identity. Repeats Phase 2's reconciliation as a built-in step. | ✓ |
| Edit project copy only, sync manually later | Smaller phase, but the live /agens runs the old version until manually synced — the drift Phase 2 had to clean up. | |
| Retire one copy | Single install location via symlink or deploy script. Structural fix, bigger than this phase needs. | |

**User's choice:** Edit project copy, redeploy to user level (the recommended option).
**Notes:** None.

---

## Claude's Discretion

- Exact wording of the fixed refusal text (follow the Phase 1 refusal register).
- Exact scoping syntax for the Bash grant in skill frontmatter — verify what `allowed-tools` specifiers actually support.

## Deferred Ideas

None — discussion stayed within phase scope.
