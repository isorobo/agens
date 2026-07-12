# Phase 2: Delegation Wiring - Discussion Log

> **Audit trail only.** Do not use as input to planning, research, or execution agents.
> Decisions are captured in CONTEXT.md — this log preserves the alternatives considered.

**Date:** 2026-07-12
**Phase:** 2-Delegation Wiring
**Areas discussed:** Delegation target & invocation, Trigger detection, Absence detection & failure message, Post-delegation handoff

---

## Delegation target & invocation

| Option | Description | Selected |
|--------|-------------|----------|
| Always call gsd-ai-integration-phase | The only real Skill-tool target. Accept its full behavior: requires a GSD phase number, writes AI-SPEC.md into a phase directory. | ✓ |
| Treat gsd-framework-selector as unreachable | Since it isn't Skill-tool-invocable, agens never targets it directly; declines per DELEGATE-02 if gsd-ai-integration-phase can't serve the question either | |

**User's choice:** Always call gsd-ai-integration-phase.
**Notes:** Scouting surfaced that `gsd-framework-selector` has no Skill-tool entry point — it's an Agent-only subagent spawned internally by `gsd-ai-integration-phase`.

| Option | Description | Selected |
|--------|-------------|----------|
| Gate on active GSD phase | Check for `.planning/` and a current phase before offering delegation; decline with a clear message if absent | ✓ |
| Attempt regardless, let target handle it | Always invoke; let the target skill's own argument-hint/prompt handle a missing phase number | |

**User's choice:** Gate on active GSD phase.
**Notes:** —

| Option | Description | Selected |
|--------|-------------|----------|
| Pass agens' answers as context | Include the four questionnaire answers in the Skill invocation as background context | ✓ |
| No handoff — clean separation | gsd-ai-integration-phase runs its own independent intake | |

**User's choice:** Pass agens' answers as context.
**Notes:** —

| Option | Description | Selected |
|--------|-------------|----------|
| Pre-check via Glob before invoking | Glob for the skill directory before calling the Skill tool; emit fixed failure message if absent | ✓ |
| Invoke and catch failure reactively | Call the Skill tool directly and translate any error into the fixed failure message | |

**User's choice:** Pre-check via Glob before invoking.
**Notes:** User initially dismissed a follow-up round (phase-number selection + failure-message wording) mid-discussion, then clarified they meant to discuss the other three gray areas as well — the area itself (target & invocation) was already settled before that dismissal.

---

## Trigger detection

| Option | Description | Selected |
|--------|-------------|----------|
| Keyword/semantic match on framework names | Detect via known SDK/framework terms or phrasing like "which framework/SDK should I use" | ✓ |
| Add a fifth questionnaire dimension | Extend the fixed intake with an explicit framework-guidance question | |
| Two separate entry points | Pattern recommendation and framework-fit are two distinct SKILL.md triggers | |

**User's choice:** Keyword/semantic match on framework names.
**Notes:** —

| Option | Description | Selected |
|--------|-------------|----------|
| Detected upfront, before intake starts | Check the initial message first; if matched, skip the pattern questionnaire and go straight to delegation | ✓ |
| Detected any time, including mid-recommendation | A follow-up framework-fit question re-triggers detection even after intake/recommendation has started | |

**User's choice:** Detected upfront, before intake starts.
**Notes:** Mid-conversation detection was explicitly deferred (see CONTEXT.md Deferred Ideas).

---

## Absence detection & failure message

| Option | Description | Selected |
|--------|-------------|----------|
| Name + reason + next step | States what's missing, why agens is stopping, and what would fix it | ✓ |
| Name + reason only, no fix instruction | States the fact and agens' router identity, leaves the fix to the user | |

**User's choice:** Name + reason + next step.
**Notes:** —

| Option | Description | Selected |
|--------|-------------|----------|
| One combined pre-flight check | Check skill presence AND active-phase presence together; one message names whichever condition(s) failed | ✓ |
| Two separate checks, distinct messages | Skill-absent and no-active-phase are different failure modes with different fixes — keep separate messages | |

**User's choice:** One combined pre-flight check.
**Notes:** —

---

## Post-delegation handoff

| Option | Description | Selected |
|--------|-------------|----------|
| Step back — target skill owns the output | agens invokes the Skill tool then gets out of the way; the target's output is presented as-is | ✓ |
| Relay with a short framing note | agens adds a one-line intro before the target's output, but doesn't alter it | |

**User's choice:** Step back — target skill owns the output.
**Notes:** —

---

## Claude's Discretion

None — all four selected gray areas reached explicit user decisions.

## Deferred Ideas

- Mid-conversation trigger detection (framework-fit question surfacing after pattern intake/recommendation has already started) — deferred, revisit if this proves limiting.
- Phase-number selection when `STATE.md`'s current phase may be ambiguous — not settled either way; left as an open question for research/planning.
