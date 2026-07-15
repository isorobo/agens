# Phase 2: Delegation Wiring - Pattern Map

**Mapped:** 2026-07-12
**Files analyzed:** 3 (1 modified, 2 conditional new)
**Analogs found:** 3 / 3

## Orientation

Phase 2 is a wiring phase. It edits one file — `.claude/skills/agens/SKILL.md` —
and installs nothing. The work adds one delegation branch: detect a framework-fit
question on the user's first message, run a combined pre-flight gate, then invoke
`gsd-ai-integration-phase` via the `Skill` tool or refuse with a fixed message.

Every pattern the branch needs already exists on disk. Three analog classes cover
it:

1. **A router skill that dispatches via the `Skill` tool** — `gsd-ns-workflow/SKILL.md`.
   It grants exactly `Read` + `Skill`, holds an intent-to-target table, and invokes
   the matched skill "directly using the Skill tool." This is the closest analog for
   the D-01 invocation and the `Skill` grant.
2. **The delegation target's own contract** — `gsd-ai-integration-phase/SKILL.md`.
   It names the `argument-hint`, the `$ARGUMENTS` binding, and inline execution that
   makes D-09 (present as-is) the default.
3. **agens' own Phase 1 body** — `.claude/skills/agens/SKILL.md`. Its gate-before-showing
   citation check (Step 2b), its resolve-first-refuse-plainly path rule, and its fixed
   refusal block are the in-repo precedents for the combined gate (D-05), the presence
   pre-check (D-04), and the fixed failure message (D-06).

**Analog files (all read for this map):**
- `C:/Users/Simon/.claude/skills/gsd-ns-workflow/SKILL.md` — router that dispatches to a matched skill via the `Skill` tool. Closest analog for D-01 and the `Skill` grant.
- `C:/Users/Simon/.claude/skills/gsd-ai-integration-phase/SKILL.md` — the delegation target: frontmatter, `argument-hint`, `$ARGUMENTS`, inline execution.
- `C:/Users/Simon/claude-projects/agens/.claude/skills/agens/SKILL.md` — agens' Phase 1 body, the file Phase 2 edits and the source of the gate/refusal/path-resolution shapes.
- `C:/Users/Simon/claude-projects/agens/.planning/STATE.md` — the phase-gate read target (D-02), `## Current Position → Phase:`.

## File Classification

| New/Modified File | Role | Data Flow | Closest Analog | Match Quality |
|-------------------|------|-----------|----------------|---------------|
| `.claude/skills/agens/SKILL.md` (modified) | config (skill definition + orchestration body) | request-response + routing branch (detect → gate → delegate/refuse) | `gsd-ns-workflow/SKILL.md` (router-dispatch) + agens Phase 1 body (gate/refusal) | role-match (strong) |
| `.claude/skills/agens/references/trigger-terms.md` (conditional new) | config (reference) | transform (user message → framework-fit match) | `eulogy/SKILL.md` references pattern (per Phase 1 map) | role-match |
| `.claude/skills/agens/references/delegation-refusal.md` (conditional new) | config (reference) | request-response (fixed failure template) | `eulogy/SKILL.md` references pattern (per Phase 1 map) | role-match |

**Conditional files:** the two `references/*.md` files exist only if `SKILL.md`
nears the 500-line guidance after the branch lands (RESEARCH.md §Recommended
Project Structure, §Supporting). The current `SKILL.md` is 168 lines; the branch
is unlikely to breach 500, so the default is one self-contained `SKILL.md`. The
planner should treat the references split as a size-triggered option, not a
required deliverable. Note: `references/` already holds `trigger-tests.md` (a test
harness, unrelated) and `agent-authored-convention.md` (Phase 4 convention) — do
not confuse `trigger-tests.md` with a new `trigger-terms.md`.

## Pattern Assignments

### `.claude/skills/agens/SKILL.md` (config, request-response + routing branch)

Four edits land in this file (RESEARCH.md Q4 insertion points). Each maps to a
concrete analog excerpt below.

---

**Edit 1 — Frontmatter: add `Skill` to `allowed-tools`** (agens SKILL.md line 12)

Current grant, to be extended:
```yaml
allowed-tools: Read Grep Glob AskUserQuestion
```
Target: `allowed-tools: Read Grep Glob AskUserQuestion Skill`.

**Analog:** `gsd-ns-workflow/SKILL.md` lines 1-7 — a router skill whose whole job
is Skill-tool dispatch, granting exactly the tools it needs and no more.
```yaml
---
name: gsd-ns-workflow
description: "workflow | discuss plan execute verify phase progress"
allowed-tools:
  - Read
  - Skill
---
```
Copy the principle, not the list: agens keeps its four Phase 1 read tools and adds
only `Skill`. No `Write`, `Edit`, or `Bash` (RESEARCH.md Security Domain; least
privilege, fail closed). Without this edit every delegation call prompts for
permission (RESEARCH.md Pitfall 3). This is a required edit.

---

**Edit 2 — New `## Step 0: Detect a framework-fit question`** (insert ahead of Step 1, current line 51)

**Analog for placement:** agens SKILL.md `## When to Use This Skill` (lines 39-49)
and `## Step 1` (line 51). Step 0 sits between them — detection runs on the first
message, before the four-question intake. On a match it branches to the delegation
section and skips Steps 1-3 (D-08); on no match it falls through to Step 1 unchanged.

**Analog for the routing-table shape:** `gsd-ns-workflow/SKILL.md` lines 15-27 — an
intent-to-target table plus a one-line dispatch instruction. agens' Step 0 is the
single-target degenerate case: one framework-fit intent, one target.
```markdown
| User wants | Invoke |
|---|---|
| Gather context before planning | gsd-discuss-phase |
| Create a PLAN.md | gsd-plan-phase |
| Execute plans in a phase | gsd-execute-phase |
...
Invoke the matched skill directly using the Skill tool.
```
For agens, the match is keyword/semantic on framework and SDK terminology —
LangChain, CrewAI, "Agent SDK", LangGraph, "which framework", "which SDK", "which
library should I use" (D-07, RESEARCH.md line 17). Do not add a second advertised
entry point in "When to Use" (D-07 forbids two entry points); detection lives in
Step 0 only.

---

**Edit 3 — New `## Delegate a framework-fit question` section** (insert after Step 3, before Anti-patterns, current line 160)

This section carries the combined gate (D-05), the invocation (D-01/D-03), and the
present-as-is instruction (D-09). Three sub-patterns compose it.

**3a. Combined pre-flight gate — analog: agens SKILL.md Step 2b** (lines 111-130).
Phase 1's citation gate is the exact "run every check before showing anything, any
failure routes to the plain refusal" shape. Reuse it for skill-presence + phase-gate.
```markdown
### 2b. Run the citation gate before showing anything

Both checks below run BEFORE any recommendation reaches the user. Either failure
routes to the plain refusal.

1. **Path resolves.** Read or Glob the vault-relative path
   `30_Concepts/agent-patterns-index.md`, resolved against the vault root per
   the resolution rule above. If the call errors (file not found), or returns
   a file that exists but is empty, the path does not resolve — refuse.
2. **Bold entry present.** Grep that note ONLY, matching the candidate's bold
   `**Name**` literal anchored to the bold form ...
```
For Phase 2, the two checks become (RESEARCH.md Code Examples, lines 301-318):
- **A. Target present** — Glob `${CLAUDE_SKILL_DIR}/../gsd-ai-integration-phase/SKILL.md`.
  No hit, or a path that cannot resolve, fails A (refuse; never answer inline).
- **B. Active GSD phase** — Glob `${CLAUDE_PROJECT_DIR}/.planning/STATE.md`; Read it;
  parse the leading integer on the `Phase:` line under `## Current Position`. Absent
  or unparseable fails B.

Both pass, proceed. Either fails, emit the fixed failure message naming every failed
condition at once (D-05).

**3b. Path-resolution rule — analog: agens SKILL.md lines 28-37** ("Resolving the
vault-relative path"). Phase 1's resolve-first-refuse-plainly-on-failure shape is the
precedent for anchoring the presence and state paths against environment substitutions
rather than a hard-coded root.
```markdown
**Resolving the vault-relative path.** `Read` requires an absolute path and
`Glob` searches only its `path` parameter's directory ... Before Step 2b,
determine the vault root from the session's granted additional directories ...
If no granted directory contains a `30_Concepts/` folder, the vault was not
granted this session — that is itself a "path does not resolve" failure; refuse ...
```
Apply the same shape with `${CLAUDE_SKILL_DIR}` (presence anchor) and
`${CLAUDE_PROJECT_DIR}` (state anchor). The refuse-on-uncertainty backstop is
mandatory: any unresolved path declines rather than answers inline (RESEARCH.md
Q5 resolution 2, Pitfall 1).

**3c. The Skill-tool invocation — analog: `gsd-ai-integration-phase/SKILL.md`**
(lines 1-16, 30-32) supplies the target contract; `gsd-ns-workflow/SKILL.md` line 27
supplies the dispatch instruction.

Target contract (the shape agens invokes against):
```yaml
name: gsd-ai-integration-phase
argument-hint: "[phase number]"
```
```markdown
<context>
Phase number: $ARGUMENTS — optional, auto-detects next unplanned phase if omitted.
</context>
```
The phase integer parsed in 3a check B binds to `$ARGUMENTS`. The four questionnaire
answers append as trailing background prose — best-effort context, not a guaranteed
pass-through, because the target's workflow reads the phase `CONTEXT.md`, not the
invocation text (RESEARCH.md Q2, Assumption A1, Pitfall 2). Invocation form
(RESEARCH.md Code Examples lines 286-298):
```markdown
Invoke the Skill tool with:
  skill name: gsd-ai-integration-phase
  arguments:  "<phase-integer-from-STATE.md>

  Background from agens (do not re-ask the user these):
  - goal: <goal answer>
  - workflow: <workflow answer>
  - data sensitivity: <sensitivity answer>
  - latency/cost: <latency answer>"
Then stop. Present the skill's own prompts and AI-SPEC.md output as-is. (D-09)
```
The target runs inline (no `context: fork`), so D-09 is the default: agens stops
narrating after the call and the target's prompts and `AI-SPEC.md` appear directly.

---

**Edit 4 — Fixed failure message (D-06) — analog: agens SKILL.md Step 3 refusal
block** (lines 142-158). Phase 1's fixed refusal is the precedent: state plainly
what was searched, cite nothing, offer no partial substitute.
```markdown
### Plain refusal (any gate check failed, no Trigger fits, or free-text "Other")

Emit the fixed refusal below. Name the four searched dimensions, cite nothing,
and offer no "closest" pattern.

```
No pattern in the wiki-agents vault matches this shape.

I searched agent-patterns-index.md for a pattern whose trigger fits:
- goal: {goal}
...
None resolved. I will not cite a loosely related pattern to fill the gap.
```
```
For Phase 2, swap the body for the name + reason + next-step template (RESEARCH.md
Code Examples lines 320-332). Keep the same discipline: fixed text, conditional lines
for whichever gate condition failed, no inline framework opinion.
```
I route framework and SDK questions to the GSD AI-integration skill rather than
answering them myself, and I cannot start that hand-off right now.

Blocking:
- {if A} The gsd-ai-integration-phase skill is not installed where I can find it.
- {if B} You are not inside an active GSD phase (no .planning/STATE.md current phase).

To proceed:
- {if A} Install the GSD skill family so gsd-ai-integration-phase is available.
- {if B} Start or resume a GSD phase, then ask again.
```

---

**Edit 5 — Extend `## Anti-patterns`** (agens SKILL.md lines 160-167). The existing
two entries model the shape: a bold name, then a sentence naming the failure and why
it destroys the skill's value.
```markdown
## Anti-patterns

- **Decorative citation.** Never cite the index note, or a near-miss entry, "to
  look thorough". A citation is valid only when it is a specific bold entry whose
  Trigger the four answers satisfy.
- **Un-gated model-knowledge recommendation.** Never recommend a pattern without
  the Step 2b Read+Grep gate passing ...
```
Add the four Phase 2 anti-patterns in the same form (RESEARCH.md Anti-Patterns,
lines 168-173): wrapping the target's output, reactive absence handling, answering
inline on gate failure, inventing a phase number.

---

### `.claude/skills/agens/references/*.md` (config, conditional)

Only if `SKILL.md` nears 500 lines. Analog: the `eulogy/SKILL.md` references pattern
recorded in the Phase 1 map (`01-PATTERNS.md` lines 167-188) — a lean body that names
each reference inline at the point of use and lists them at the bottom. Split the
trigger vocabulary into `references/trigger-terms.md` (D-07) and the fixed failure
template into `references/delegation-refusal.md` (D-06), referenced this way so Claude
loads them on demand.

## Shared Patterns

### `Skill`-tool router dispatch
**Source:** `gsd-ns-workflow/SKILL.md` lines 1-7 (grant) and 15-27 (dispatch)
**Apply to:** the `agens` frontmatter (`Skill` grant) and Step 0 + delegation section
A router grants `Skill`, matches user intent to a target, and invokes "the matched
skill directly using the Skill tool." agens is the single-target case of this exact
pattern. Grant only `Skill` on top of the Phase 1 read tools; add no write capability
(least privilege, fail closed — RESEARCH.md Security Domain).

### Gate-before-acting, refuse-plainly on failure
**Source:** `agens/SKILL.md` lines 111-130 (Step 2b gate) and 142-158 (fixed refusal)
**Apply to:** the combined pre-flight gate (D-05) and the fixed failure message (D-06)
Phase 1 already proves the shape: run every precondition before any user-facing action,
and on any failure emit fixed text that names what was checked and offers no dressed-up
substitute. Phase 2 reuses it for skill-presence + phase-gate, not a new failure idiom
(CONTEXT.md Established Patterns).

### Resolve-first, refuse-on-uncertainty path handling
**Source:** `agens/SKILL.md` lines 28-37 (vault-root resolution rule)
**Apply to:** the presence anchor (`${CLAUDE_SKILL_DIR}`) and state anchor (`${CLAUDE_PROJECT_DIR}`)
Resolve a path against a known environment anchor; treat a non-resolving path as a
failure and refuse plainly rather than guess. Hard-code no absolute root (Phase 1
precedent; RESEARCH.md Security Domain path-handling row). The backstop is mandatory:
an unresolved presence check declines, never answers inline (RESEARCH.md Q5).

### Delegation target contract (read-only reference)
**Source:** `gsd-ai-integration-phase/SKILL.md` lines 1-16, 30-32
**Apply to:** the Skill-tool invocation shape (D-01/D-03)
The target binds `$ARGUMENTS` to a phase number and runs inline. agens passes the
STATE.md phase integer as the deterministic argument and appends its four answers as
trailing best-effort context. agens reads this contract; it never authors the target's
frontmatter.

### Current-phase signal
**Source:** `.planning/STATE.md` lines 26-31 (`## Current Position`)
**Apply to:** the phase-gate check B and the `$ARGUMENTS` value
```
## Current Position

Phase: 2
Plan: Not started
Status: Ready to plan
```
The load-bearing value is the integer on the `Phase:` line under `## Current Position`
(RESEARCH.md Q3). Parse only the leading integer; an absent or unparseable value fails
check B — never enumerate `phases/` directories to guess (RESEARCH.md Anti-Patterns,
Don't Hand-Roll).

## No Analog Found

None for the mechanics. Every capability Phase 2 wires has a working analog on disk.

Two items are new orchestration the planner authors rather than copies, because no
skill demonstrates them:

| Concern | Why no analog | Where it comes from |
|---------|---------------|---------------------|
| Combined multi-condition gate reporting every failure at once (D-05) | Phase 1's gate has two checks but routes to one generic refusal; it does not enumerate which condition failed | RESEARCH.md Code Examples lines 301-332 — write conditional `{if A}` / `{if B}` lines |
| Deployment-location decision (agens at user level vs project level) | Not a code pattern; a plan decision that governs whether the sibling presence-check resolves | RESEARCH.md Q5, Runtime State Inventory, Assumption A2 — plan must decide, backstop protects either way |

## Metadata

**Analog search scope:**
- In-repo: `C:/Users/Simon/claude-projects/agens/.claude/skills/agens/` — the file under edit, plus its `references/`.
- On-disk skills: `C:/Users/Simon/.claude/skills/` — 8 skills using the `Skill` tool enumerated via Grep; `gsd-ns-workflow` (router dispatch) and `gsd-ai-integration-phase` (target contract) read.
- Project state: `C:/Users/Simon/claude-projects/agens/.planning/STATE.md` — `## Current Position` format read.

**Files scanned:** 4 analogs read; 1 Grep pass over `~/.claude/skills/` for `Skill`-tool precedent.
**Pattern extraction date:** 2026-07-12
