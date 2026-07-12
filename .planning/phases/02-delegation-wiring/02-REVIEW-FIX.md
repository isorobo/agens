---
phase: 02-delegation-wiring
fixed_at: 2026-07-12T00:00:00Z
review_path: .planning/phases/02-delegation-wiring/02-REVIEW.md
iteration: 1
findings_in_scope: 6
fixed: 6
skipped: 0
status: all_fixed
---

# Phase 2: Code Review Fix Report

**Fixed at:** 2026-07-12
**Source review:** .planning/phases/02-delegation-wiring/02-REVIEW.md
**Iteration:** 1

**Summary:**
- Findings in scope: 6
- Fixed: 6
- Skipped: 0

All Critical and Warning findings were resolved. The two Info findings (IN-01,
IN-02) fall outside this run's `critical_warning` scope and were not addressed.

## Fixed Issues

### CR-01: Delegation invocation requires four answers that Step 0 instructs skipping

**Files modified:** `.claude/skills/agens/SKILL.md`
**Commit:** dac3dbb
**Applied fix:** Rewrote the Delegate section's Invocation step (part 2) to make
the four-answer background block conditional and best-effort, preserving both
locked decisions from CONTEXT.md. D-08 (Step 0 skips the questionnaire) is kept
intact; D-03 (four answers passed as background when available) is honoured. The
step now branches on whether a prior Step 1 run in the same conversation collected
answers: when answers exist, it appends the labelled block; when the Step 0
framework-fit path reached delegation without an intake, it passes the phase
integer alone with no block and no placeholder tokens. Added an explicit rule
never to re-run Step 1 or fabricate answers, and a second worked argument-string
example for the no-answers path. This removes the contradiction that forced the
executing model to fabricate, emit placeholders, or re-run the forbidden intake.

### CR-02: Step 0 keyword match can misroute agens' own core pattern-recommendation job

**Files modified:** `.claude/skills/agens/SKILL.md`
**Commit:** 73b3956
**Applied fix:** Rewrote Step 0's match rule to require the "which framework / SDK
/ library should I use" QUESTION SHAPE rather than bare keyword presence. Replaced
the mixed keyword list with qualifying question phrasings ("which framework should
I use", "LangChain or CrewAI?", "is LangGraph a good fit"). Added the review's own
counter-example ("I want to build an agent using the Claude Agent SDK that
summarises documents") as an explicit non-match that falls through to Step 1, so a
product name inside a pattern-scoping sentence no longer misroutes away from
agens' core job.

### WR-01: `Skill` tool grant is unscoped; the single-target restriction is prompt-only

**Files modified:** `.claude/skills/agens/SKILL.md`
**Commit:** 79103d9
**Applied fix:** Added an anti-pattern entry documenting the residual risk in the
shipped SKILL.md (previously only in 02-CONTEXT.md's STRIDE table). It states that
`allowed-tools` cannot scope the `Skill` tool to one target, that the
single-target rule is prompt-enforced only, names the write-capable
`build-mcp-server` as the concrete escalation risk, and instructs that the
`agens-build` phase must keep its human-approval gate. This is a platform
limitation, not a fixable defect; the fix makes the risk visible where it matters.

### WR-02: Check B verifies a phase number exists, not that the phase is active

**Files modified:** `.claude/skills/agens/SKILL.md`
**Commit:** 25814fb
**Applied fix (requires human verification):** Rewrote Check B to parse both the
leading integer AND the trailing status token after the em-dash separator, and to
pass only when the status marks the phase in progress (for example `EXECUTING`,
not `COMPLETE`). A completed phase carrying a trailing integer now fails the gate;
a `Phase:` line with no status token is treated as not-active. This aligns the
check with D-02's "current/in-progress phase" intent and matches the live
STATE.md format (`Phase: 02 (delegation-wiring) — EXECUTING`). This is a logic
change to gate behaviour — a human should confirm the status-token vocabulary
(EXECUTING / COMPLETE and any other values GSD emits) is handled correctly before
the phase proceeds.

### WR-03: Ambiguous instruction for stripping `{if A}`/`{if B}` template markers

**Files modified:** `.claude/skills/agens/SKILL.md`
**Commit:** 825691b
**Applied fix:** Replaced the internally inconsistent "emit this text exactly,
keeping only the `{if A}`/`{if B}` lines" instruction with an explicit rule to
drop the literal `{if A}` / `{if B}` prefix tokens, keep only the sentence for the
failed condition, and delete lines for any passed condition. Added three worked
cases (only A failed, only B failed, both failed) so the raw marker text never
reaches the user.

### WR-04: Auto-trigger `description` was not extended for framework/SDK phrasing

**Files modified:** `.claude/skills/agens/SKILL.md`
**Commit:** 25ae55c
**Applied fix:** Extended the frontmatter `description` with representative
framework-fit question phrasing ("which framework should I use", "LangChain or
CrewAI?", "is LangGraph a good fit") so auto-invocation reaches Step 0's router for
framework-fit openers, not only pattern-fit ones. The added phrasings use the
question shape mandated by the CR-02 fix, keeping the trigger vocabulary
consistent. Description length is 794 characters, well under the 1,536-char cap.

## Skipped Issues

None. All in-scope findings were fixed.

---

_Fixed: 2026-07-12_
_Fixer: Claude (gsd-code-fixer)_
_Iteration: 1_
