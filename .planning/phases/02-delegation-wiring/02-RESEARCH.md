# Phase 2: Delegation Wiring - Research

**Researched:** 2026-07-12
**Domain:** Claude Code Skill authoring — cross-skill delegation via the `Skill` tool, pre-flight gating, plain failure
**Confidence:** HIGH

<user_constraints>
## User Constraints (from CONTEXT.md)

### Locked Decisions
- **D-01:** agens always calls `gsd-ai-integration-phase` via the `Skill` tool for framework-fit questions. It never targets `gsd-framework-selector` directly (no Skill entry point exists for it) and never reaches for the `Agent` tool as a workaround to invoke it in isolation.
- **D-02:** Delegation is gated on the user being inside an active GSD phase. Before invoking, agens checks for `.planning/` and a current/in-progress phase (per `STATE.md`). If that precondition is not met, agens declines — see D-05.
- **D-03:** agens passes its own four questionnaire answers (goal, workflow shape, data sensitivity, latency/cost) into the `gsd-ai-integration-phase` invocation as background context, so the user is not asked a re-shaped version of the same questions twice.
- **D-04:** Presence checking is a pre-check, not a reactive catch. agens Globs for the `gsd-ai-integration-phase` skill directory before ever attempting a `Skill` tool call.
- **D-05:** The pre-flight check is combined: it verifies BOTH skill presence (D-04) AND the active-GSD-phase gate (D-02) before any invocation attempt. One failure message names whichever condition(s) failed.
- **D-06:** The fixed failure message format is name + reason + next step: state what is missing, why agens is stopping (it routes framework-fit questions rather than answering inline), and what would fix it.
- **D-07:** agens distinguishes a framework-fit question from its pattern-recommendation job by keyword/semantic match on framework and SDK terminology (LangChain, CrewAI, "Agent SDK", LangGraph, "which framework", "which SDK", "which library should I use") — not a fifth questionnaire dimension, and not two advertised entry points.
- **D-08:** Detection happens upfront, on the user's initial message, before the four-question pattern intake starts. If matched, agens skips the pattern questionnaire and goes straight to the delegation flow. Mid-conversation framework-fit language is out of scope.
- **D-09:** Once delegation succeeds, agens steps back. `gsd-ai-integration-phase`'s own output is presented as-is — not paraphrased, summarised, or re-framed.

### Claude's Discretion
- Nothing was explicitly deferred to Claude's discretion this session — all four selected gray areas reached explicit user decisions.

### Deferred Ideas (OUT OF SCOPE)
- **Mid-conversation trigger detection** — a framework-fit question surfacing after the pattern questionnaire has started, or after a recommendation has been given, is out of scope (D-08 restricts detection to the user's initial message).
- **Phase-number selection when multiple GSD phases are candidates** — researched below; see Open Questions for the recommendation.
</user_constraints>

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|------------------|
| DELEGATE-01 | User's framework-fit questions are routed to `gsd-framework-selector`/`gsd-ai-integration-phase` via the `Skill` tool rather than answered inline. | Skill-tool invocation contract verified (Standard Stack, Code Examples). `gsd-ai-integration-phase` is the only Skill-reachable target; `gsd-framework-selector` is Agent-tool-only. Invocation flow specified in Architecture Patterns. |
| DELEGATE-02 | User sees an explicit, clear failure message — never a silent inline reimplementation — when a delegated Skill is not installed or not found. | Combined pre-flight gate (Glob presence + STATE.md phase gate) and fixed name/reason/next-step message specified. Refuse-on-uncertainty posture backstops the presence check (Common Pitfalls). |
</phase_requirements>

## Summary

Phase 2 adds one delegation branch to the existing `agens/SKILL.md`. When the user's opening message names a framework or SDK, agens skips its four-question pattern intake and hands the question to `gsd-ai-integration-phase` through the `Skill` tool. Before the hand-off, agens runs one combined pre-flight check — the target skill is present (Glob) AND the user sits inside an active GSD phase (`.planning/STATE.md`) — and declines with a fixed name/reason/next-step message when either condition fails. This is a router boundary, not a feature: agens never answers a framework question from its own knowledge.

Three facts shape the plan. First, the `Skill` tool passes arguments through `$ARGUMENTS`, and `gsd-ai-integration-phase` binds `$ARGUMENTS` to a phase number — so the phase number flows deterministically, but the four questionnaire answers (D-03) have no input slot in the target's workflow and reach the sub-agents only as best-effort conversation context. Second, `gsd-ai-integration-phase` runs inline (no `context: fork`), so its own prompts and `AI-SPEC.md` output appear directly to the user, which makes D-09 (present as-is) the natural outcome rather than extra work. Third, agens' current `allowed-tools` grant omits `Skill`; the plan must add it, or every delegation call prompts for permission.

One deployment fact is load-bearing and needs a decision in the plan. agens ships in the repo at project level (`.claude/skills/agens/`), while the target ships at user level (`~/.claude/skills/gsd-ai-integration-phase/`). The two are not siblings, so the clean portable presence-check anchor (`${CLAUDE_SKILL_DIR}/../gsd-ai-integration-phase/SKILL.md`) only resolves when agens itself runs from its user-level install — the install CLAUDE.md already names as canonical.

**Primary recommendation:** Add a Step 0 trigger-detection branch ahead of Step 1, gated by a combined Glob-presence + STATE.md-phase pre-flight check, that invokes `gsd-ai-integration-phase` via the `Skill` tool with the STATE.md current-phase number as `$ARGUMENTS` and the four answers appended as background context; grant the `Skill` tool; deploy agens at user level so sibling presence-resolution holds; and adopt refuse-on-uncertainty as the standing backstop.

## Architectural Responsibility Map

| Capability | Primary Tier | Secondary Tier | Rationale |
|------------|-------------|----------------|-----------|
| Framework-fit trigger detection | agens `SKILL.md` body (orchestration) | — | Detection is a routing judgement over the user's first message; it belongs in agens' instructions, not in any tool. |
| Skill-presence pre-check | `Glob` tool | agens body | Deterministic file existence check; agens body interprets the result and decides to proceed or refuse. |
| Active-GSD-phase gate | `Read`/`Glob` over `.planning/STATE.md` | agens body | The gate reads project state from a known file; the body parses the current-phase signal. |
| Framework-fit answer (the delegated judgement) | `gsd-ai-integration-phase` (via `Skill` tool) | its sub-agents (`gsd-framework-selector` et al.) | The whole point of the phase — agens owns none of this judgement; it triggers the target and steps back. |
| Failure messaging | agens body | — | Fixed refusal text is agens' own responsibility; it fires before any delegation attempt. |

## Standard Stack

### Core

| Mechanism | Where | Purpose | Why Standard |
|-----------|-------|---------|--------------|
| `Skill` tool | agens `allowed-tools` + body | Invoke `gsd-ai-integration-phase` from inside agens | The sanctioned cross-skill invocation mechanism; CLAUDE.md already names it for the MCP Builder hand-off. `[CITED: code.claude.com/docs/en/skills]` |
| `$ARGUMENTS` substitution | the invocation call | Pass the phase number to the target skill | The target's `argument-hint` is `[phase number]` and its body binds `$ARGUMENTS` to the phase number. `[VERIFIED: on-disk SKILL.md]` |
| `Glob` tool | agens body | Presence pre-check for the target skill directory (D-04) | Already granted in Phase 1; deterministic existence check. `[VERIFIED: Phase 1 SKILL.md]` |
| `Read` / `Glob` over `.planning/STATE.md` | agens body | Active-phase gate (D-02) and phase-number source | STATE.md is GSD's single source of current position. `[VERIFIED: state template + project STATE.md]` |
| `${CLAUDE_PROJECT_DIR}` substitution | agens body | Portable anchor to the user's project root for the `.planning/` check | Resolves the project root independent of install location. `[CITED: code.claude.com/docs/en/skills]` |
| `${CLAUDE_SKILL_DIR}` substitution | agens body | Portable anchor for the sibling presence-check path | Resolves the skill's own directory regardless of install level. `[CITED: code.claude.com/docs/en/skills]` |

### Supporting

| Mechanism | Where | Purpose | When to Use |
|-----------|-------|---------|-------------|
| `references/*.md` split | skill dir | Move the trigger vocabulary and failure template out of `SKILL.md` | Only if `SKILL.md` nears the 500-line guidance after the new branch lands. `[CITED: code.claude.com/docs/en/skills]` |
| `argument-hint` (read, not authored) | target skill | Confirms the target's expected input shape | Read-only reference — agens does not author this. `[VERIFIED: on-disk SKILL.md]` |

### Alternatives Considered

| Instead of | Could Use | Tradeoff |
|------------|-----------|----------|
| `Skill` tool to invoke `gsd-ai-integration-phase` | `Agent` tool to spawn `gsd-framework-selector` directly | Forbidden by D-01. `gsd-framework-selector` has no Skill entry point; reaching it via `Agent` reimplements the target's own orchestration and drops its gates. |
| Passing the phase number explicitly | Omitting `$ARGUMENTS`, letting the target auto-detect the next unplanned phase | Auto-detect can pick a different phase than the user's current one and drops D-02's confirmation intent. Pass explicitly. |
| Glob presence pre-check | Relying on the Skill tool erroring when the skill is absent | Violates D-04 (pre-check, not reactive catch). A reactive catch risks a silent inline answer before the error surfaces. |
| Deterministic Glob presence-check | Trusting Claude's loaded skill-descriptions to know the target exists | D-04 demands an explicit pre-check. Use Claude's awareness only as a secondary signal, never as the sole gate. |

**Installation:** No package manager. Phase 2 edits one markdown file (`.claude/skills/agens/SKILL.md`) and optionally adds `references/*.md`. No dependency is installed, so no Package Legitimacy Audit applies.

## Architecture Patterns

### System Architecture Diagram

```
User's initial message
        │
        ▼
┌─────────────────────────────┐
│ Step 0: trigger detection   │  keyword/semantic match on framework/SDK terms (D-07)
│ (agens body, on first msg)  │  — LangChain, CrewAI, "Agent SDK", "which framework"…
└─────────────┬───────────────┘
              │
       framework-fit? ──── NO ──► existing Step 1 (four-question pattern intake, unchanged)
              │ YES (D-08: skip pattern intake)
              ▼
┌─────────────────────────────────────────────┐
│ Combined pre-flight gate (D-05, ONE check)  │
│  A. Glob target skill dir present? (D-04)   │
│  B. .planning/ + STATE.md current phase?    │  reads ${CLAUDE_PROJECT_DIR}/.planning/STATE.md (D-02)
└───────┬─────────────────────────────┬───────┘
        │ both pass                    │ either fails
        ▼                              ▼
┌────────────────────────┐   ┌──────────────────────────────────┐
│ Skill tool invocation  │   │ Fixed failure message (D-06)      │
│  target: gsd-ai-       │   │  name + reason + next step,        │
│  integration-phase     │   │  listing every failing condition   │
│  $ARGUMENTS = phase N  │   │  at once (D-05). No inline answer. │
│  + 4 answers as        │   └──────────────────────────────────┘
│  background (D-03)      │
└───────────┬────────────┘
            │ runs inline (no context: fork)
            ▼
┌────────────────────────────────────────────┐
│ Target's own prompts + AI-SPEC.md output    │  agens presents as-is (D-09), no paraphrase
│ appear directly to the user                 │
└────────────────────────────────────────────┘
```

### Recommended Project Structure

```
.claude/skills/agens/
├── SKILL.md                 # add: Skill to allowed-tools; Step 0 branch; delegation flow
└── references/              # optional, only if SKILL.md nears 500 lines
    ├── trigger-terms.md     # the framework/SDK vocabulary for D-07
    └── delegation-refusal.md# the fixed name/reason/next-step template for D-06
```

### Pattern 1: Combined pre-flight gate before a side-effecting call

**What:** Run every precondition as one check and report all failures together, before attempting the delegated call.
**When to use:** D-05 — skill presence AND active-phase gate.
**Example:**
```markdown
Before any Skill tool call, verify BOTH:
  A. Presence — Glob the target skill's SKILL.md at the resolved path. Missing → fail A.
  B. Active phase — Read ${CLAUDE_PROJECT_DIR}/.planning/STATE.md; parse the integer
     on the "Phase:" line under "## Current Position". Absent/unparseable → fail B.
If A or B fails, emit the fixed failure message naming every failed condition. Do not invoke.
If both pass, proceed to the Skill tool call.
```
This mirrors Phase 1's "gate-before-showing-anything" citation check (SKILL.md Step 2b), applied to delegation.

### Pattern 2: Deterministic argument, best-effort context

**What:** Pass the load-bearing value as a clean positional argument; append soft context as trailing prose.
**When to use:** The `Skill` call — phase number is deterministic (D-02), the four answers are background (D-03).
**Example:**
```markdown
Invoke the Skill tool:
  skill: gsd-ai-integration-phase
  arguments: "<phase-number>

  Background from agens (do not re-ask): goal=<goal>; workflow=<workflow>;
  sensitivity=<sensitivity>; latency=<latency>."
```
The target extracts the phase number from `$ARGUMENTS`; the background text sits in the shared conversation for the target's orchestrator to use if it chooses.

### Anti-Patterns to Avoid

- **Wrapping the target's output.** Never summarise or re-frame `gsd-ai-integration-phase`'s prompts or `AI-SPEC.md` (breaks D-09). It runs inline; let it speak.
- **Reactive absence handling.** Never call the `Skill` tool first and catch the error (breaks D-04). Pre-check with Glob.
- **Answering inline on gate failure.** A failed gate routes to the fixed refusal, never to agens' own framework opinion (the core DELEGATE-02 failure mode).
- **Inventing a phase number.** Never enumerate phase directories to guess a phase. Read the single current-phase value from STATE.md, or fail loud.

## Don't Hand-Roll

| Problem | Do Not Build | Use Instead | Why |
|---------|--------------|-------------|-----|
| Framework selection judgement | An inline "which framework fits" answer | `gsd-ai-integration-phase` via `Skill` tool | The delegation constraint; an inline copy drifts from the source it exists to route to. |
| Reaching `gsd-framework-selector` | An `Agent`-tool spawn of the selector alone | Invoke `gsd-ai-integration-phase`, which orchestrates the selector | The selector is Agent-tool-only and expects its parent workflow's gates and inputs; isolating it drops them. |
| Cross-skill invocation plumbing | A custom hand-off convention | The `Skill` tool's `$ARGUMENTS` contract | The sanctioned, documented mechanism; a bespoke convention has no runtime support. |
| Current-phase detection | A scan of `phases/` directories | The `Phase:` line under `## Current Position` in STATE.md | STATE.md is GSD's single source of position; a directory scan reintroduces the ambiguity D-02 avoids. |

**Key insight:** Phase 2 is a wiring phase. Every capability it touches already exists in a GSD skill or a Claude Code primitive; the work is routing to them cleanly and failing loudly, not building anything.

## Specific Research Findings

### Q1 — `gsd-ai-integration-phase` frontmatter and invocation contract `[VERIFIED: on-disk SKILL.md]`

`~/.claude/skills/gsd-ai-integration-phase/SKILL.md`:
- `name: gsd-ai-integration-phase`
- `description: "Generate an AI-SPEC.md design contract for phases that involve building AI systems."`
- `argument-hint: "[phase number]"`
- `allowed-tools:` Read, Write, Bash, Glob, Grep, **Agent**, WebFetch, WebSearch, AskUserQuestion, `mcp__context7__*`
- Body: `Phase number: $ARGUMENTS — optional, auto-detects next unplanned phase if omitted.`
- No `context: fork` — the skill runs **inline**, in agens' own conversation context.
- It orchestrates `gsd-framework-selector → gsd-ai-researcher → gsd-domain-researcher → gsd-eval-planner` and writes `AI-SPEC.md` into the phase directory.

Invocation contract for agens: call the `Skill` tool with `gsd-ai-integration-phase` and pass the phase number as the argument. Arguments reach the target through `$ARGUMENTS`; if a skill is invoked with arguments but omits `$ARGUMENTS`, Claude Code appends `ARGUMENTS: <value>` — but this target does use `$ARGUMENTS`, so the phase number binds directly. `[CITED: code.claude.com/docs/en/skills]`

### Q2 — What agens hands off to `[VERIFIED: workflows/ai-integration-phase.md]`

`~/.claude/get-shit-done/workflows/ai-integration-phase.md` is a 12-step orchestration that:
- Initialises from `gsd-sdk query init.plan-phase "$PHASE"`, requiring a resolvable phase.
- Reads the phase's `CONTEXT.md` and `REQUIREMENTS.md` and passes those file paths to the spawned sub-agents.
- Spawns four sub-agents via the `Agent` tool, copies the `AI-SPEC.md` template into the phase directory, and validates completeness.
- Presents its own `AskUserQuestion` gates (e.g. the "Existing AI-SPEC" prompt) directly to the user.

Consequence for D-03: the workflow consumes the phase's `CONTEXT.md`, not free-form invocation text. The four agens answers have **no defined input slot**. Because the target runs inline, the answers remain in the shared conversation and the orchestrator *may* carry them into sub-agent prompts, but the workflow does not require it. Treat D-03 as best-effort context, not a guaranteed pass-through — see Assumptions Log A1 and Open Questions.

Consequence for D-09: the workflow's prompts and `AI-SPEC.md` write are the target's own visible output. Presenting them as-is is the default when agens simply stops narrating after the `Skill` call.

### Q3 — How STATE.md represents the current phase `[VERIFIED: state template + project STATE.md]`

Two representations exist; the body section is the one to read:
- **Frontmatter:** `status:` (e.g. `planning`), `progress.completed_phases`, `progress.total_phases`. Machine-readable but coarse — it does not name the active phase number directly.
- **Body `## Current Position`:** the template form is `Phase: [X] of [Y] ([Phase name])` and `Status: [Ready to plan / Planning / Ready to execute / In progress / Phase complete]`. This project's live STATE.md carries `Phase: 2` and `Status: Ready to plan`.

The load-bearing signal for both D-02's gate and the phase-number argument is the **integer on the `Phase:` line under `## Current Position`**. Recommended gate:
1. `.planning/` exists (Glob `${CLAUDE_PROJECT_DIR}/.planning/STATE.md`).
2. STATE.md exists and the `## Current Position → Phase:` line yields an integer.
3. Both true → inside an active GSD phase; pass that integer to the target. Either false → decline (D-05/D-06).

`Status` can refine the gate (e.g. exclude a between-milestones state), but the phase integer is the primary and sufficient signal, and it is also the value agens must pass onward.

### Q4 — agens Phase 1 SKILL.md structure and insertion points `[VERIFIED: on-disk SKILL.md]`

Current structure of `.claude/skills/agens/SKILL.md`:
- Frontmatter (lines 1-13): `name: agens`, auto-trigger `description`, `allowed-tools: Read Grep Glob AskUserQuestion`.
- Intro prose + vault-root resolution rule (lines 15-37).
- `## When to Use This Skill` (line 39).
- `## Step 1: Ask the four fixed questions in one call` (line 51).
- `## Step 2: Match the answers to a candidate pattern, then verify` with 2a/2b (line 92).
- `## Step 3: Recommend one pattern, or refuse plainly` (line 132).
- `## Anti-patterns` (line 160).

Insertion points:
1. **Frontmatter:** add `Skill` to `allowed-tools` → `allowed-tools: Read Grep Glob AskUserQuestion Skill`. Without this grant, each delegation call prompts for permission, breaking the frictionless house style. This is a required edit.
2. **New `## Step 0: Detect a framework-fit question` before Step 1** (insert around line 50, ahead of current Step 1): the D-07 keyword/semantic match on the user's first message. On a match, branch to the delegation flow and skip Steps 1-3 (D-08). On no match, fall through to the existing Step 1 unchanged.
3. **New `## Delegate a framework-fit question` section** (insert after Step 3 or before Anti-patterns): the combined pre-flight gate (D-05), the `Skill` invocation (D-01/D-03), and the present-as-is instruction (D-09), plus the fixed failure message (D-06).
4. **`## Anti-patterns`:** extend with the four delegation anti-patterns listed above.
5. **`## When to Use This Skill`:** leave the existing pattern-recommendation phrasings; do **not** add a second advertised "framework-fit" entry point (D-07 forbids two entry points). Trigger detection lives in Step 0, not in the "When to Use" list.

### Q5 — Existing Glob presence-check pattern to mirror `[VERIFIED: Phase 1 SKILL.md] + [ASSUMED] on the sibling-path resolution`

No codebase precedent checks for a *skill's* presence. The closest analog is Phase 1's vault-root resolution (SKILL.md lines 28-37): resolve a path against a known anchor from the environment, and refuse plainly when it does not resolve. Mirror that shape for D-04:
- **Anchor:** `${CLAUDE_SKILL_DIR}` resolves to agens' own skill directory regardless of install level. `[CITED: code.claude.com/docs/en/skills]`
- **Sibling path (clean case):** `${CLAUDE_SKILL_DIR}/../gsd-ai-integration-phase/SKILL.md`. Glob it; a hit means present, a miss means absent → fail A.

**Critical deployment finding.** This sibling path only resolves when agens runs from its **user-level** install (`~/.claude/skills/agens/`), because the target ships at user level (`~/.claude/skills/gsd-ai-integration-phase/`). Verified on this machine: agens ships **project-level** at `C:/Users/Simon/claude-projects/agens/.claude/skills/agens/`, and the target is **not** a sibling of that copy. When agens loads from the project copy, `${CLAUDE_SKILL_DIR}/../gsd-ai-integration-phase` points into the project's `.claude/skills/`, where the target does not exist — a false-negative presence result.

Resolutions, in order of preference:
1. **Deploy agens at user level** (`~/.claude/skills/agens/`), the install CLAUDE.md already names canonical ("One `~/.claude/skills/agens/SKILL.md`"). Sibling resolution then holds and is fully portable. Keep the repo copy as the authoring source. This is the recommended plan decision.
2. **Refuse-on-uncertainty backstop (always include).** If the presence Glob cannot resolve the target for any reason, agens declines with the D-06 message rather than answering inline. This satisfies DELEGATE-02 even when path resolution is imperfect, and it is the standing guarantee against silent reimplementation.

There is no documented `${HOME}` or `${CLAUDE_CONFIG_DIR}` substitution, and agens holds no `Bash` grant, so a project-level agens cannot cleanly compute the user-level skills path. This makes resolution (1) the clean fix and (2) the mandatory safety net.

## Common Pitfalls

### Pitfall 1: Presence check silently resolves to the wrong directory
**What goes wrong:** `${CLAUDE_SKILL_DIR}/../gsd-ai-integration-phase` resolves under the project `.claude/skills/` (project-level agens) and reports the target absent when it is installed at user level.
**Why it happens:** agens and the target live at different install levels; the sibling assumption fails.
**How to avoid:** Deploy agens at user level (sibling resolution holds), and always keep the refuse-on-uncertainty backstop so a mis-resolution declines rather than mis-answers.
**Warning signs:** Delegation always refuses with "skill not installed" even though `/gsd-ai-integration-phase` appears in the `/` menu.

### Pitfall 2: The four answers do not reach the sub-agents
**What goes wrong:** The plan assumes D-03 passes the four answers into `gsd-framework-selector`, but the target's workflow reads the phase `CONTEXT.md`, not the invocation text.
**Why it happens:** `$ARGUMENTS` binds only the phase number; the workflow has no slot for agens' answers.
**How to avoid:** Append the answers as clearly-labelled background prose in the argument string, and set the expectation (in the plan and to the user) that they are best-effort context, not a guaranteed skip of the target's own questions.
**Warning signs:** The framework selector re-asks goal/workflow/sensitivity/latency despite agens having them.

### Pitfall 3: Missing `Skill` grant turns every delegation into a permission prompt
**What goes wrong:** agens invokes the `Skill` tool but `allowed-tools` omits it, so each call stops for approval.
**Why it happens:** Phase 1 granted only `Read Grep Glob AskUserQuestion`.
**How to avoid:** Add `Skill` to `allowed-tools` in the frontmatter edit.
**Warning signs:** A permission prompt appears on every framework-fit routing.

### Pitfall 4: Trigger detection fires mid-conversation
**What goes wrong:** agens routes to delegation after the pattern questionnaire has begun.
**Why it happens:** The detection instruction is not scoped to the first message.
**How to avoid:** Scope Step 0 explicitly to the user's initial message (D-08); state that later framework-fit language is out of scope this phase.
**Warning signs:** A pattern recommendation is abandoned partway to route a framework question.

## Code Examples

### Skill-tool invocation with phase number plus background `[CITED: code.claude.com/docs/en/skills]`
```markdown
# Inside the delegation flow, after the combined gate passes:
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

### Combined pre-flight gate `[VERIFIED: state template + Phase 1 gate pattern]`
```markdown
## Delegate a framework-fit question

Run BOTH checks before any Skill call. Report every failure at once.

A. Target present:
   Glob ${CLAUDE_SKILL_DIR}/../gsd-ai-integration-phase/SKILL.md
   No hit → condition A failed. If the path cannot resolve, treat as failed (refuse, do not answer inline).

B. Active GSD phase:
   Glob ${CLAUDE_PROJECT_DIR}/.planning/STATE.md — no hit → condition B failed.
   Read it; find the "Phase:" line under "## Current Position"; parse the leading integer.
   No integer → condition B failed.

If A or B failed, emit the fixed failure message below. Do NOT invoke and do NOT answer inline.
Otherwise invoke gsd-ai-integration-phase with the parsed phase integer.
```

### Fixed failure message (D-06: name + reason + next step)
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

## Runtime State Inventory

Phase 2 edits one markdown file and installs nothing. No stored data, live-service config, OS-registered state, secrets, or build artefacts carry a renamed string.

| Category | Items Found | Action Required |
|----------|-------------|------------------|
| Stored data | None — verified: no datastore is touched. | none |
| Live service config | None — verified: no external service configuration changes. | none |
| OS-registered state | None — verified: no OS registration. | none |
| Secrets/env vars | None — verified: no secret or env var is read or renamed. | none |
| Build artifacts | **Deployment copy**: agens should run from `~/.claude/skills/agens/` (user level) for sibling presence-resolution; the repo copy at `.claude/skills/agens/` is the authoring source. | Plan the deployment location decision (see Q5). |

## Environment Availability

| Dependency | Required By | Available | Version | Fallback |
|------------|------------|-----------|---------|----------|
| `gsd-ai-integration-phase` skill | D-01 delegation target | ✓ (user level: `~/.claude/skills/gsd-ai-integration-phase/`) | SKILL.md present, uses `$ARGUMENTS` | Combined gate refuses with D-06 message when absent |
| `Skill` tool | D-01 invocation | ✓ (Claude Code primitive) | v2.1.199+ for skill-stacking; base Skill tool current | none needed |
| `${CLAUDE_SKILL_DIR}` / `${CLAUDE_PROJECT_DIR}` substitution | presence + phase-gate anchors | ✓ | Requires Claude Code v2.1.196+ | none needed |
| `.planning/STATE.md` in the user's project | D-02 gate | ✓ in a GSD project; ✗ otherwise | GSD state format | Gate correctly declines when absent |

**Missing dependencies with no fallback:** none.
**Missing dependencies with fallback:** the target skill and an active GSD phase are both handled by the combined gate — absence produces a clean refusal, which is the intended behaviour, not a blocker.

## Security Domain

This phase authors instructions for a read-plus-delegate skill. It grants the `Skill` tool but no write, edit, or shell capability. The relevant controls:

| Concern | Applies | Standard Control |
|---------|---------|------------------|
| V5 Input validation | yes | Parse only the leading integer from the STATE.md `Phase:` line; treat an unparseable value as a gate failure, not a guessed default. Match the trigger vocabulary against the user's message text only. |
| Least privilege (fail closed) | yes | `allowed-tools` gains only `Skill`; no write/edit/Bash. The combined gate fails closed — uncertainty declines rather than proceeds (refuse-on-uncertainty). |
| No silent capability escalation | yes | agens never reimplements framework judgement; it routes or refuses. This is the DELEGATE-02 boundary and a direct mitigation of the self-mutation risk pattern PROJECT.md names. |
| Path handling | yes | Resolve presence and state paths against `${CLAUDE_SKILL_DIR}` / `${CLAUDE_PROJECT_DIR}` anchors; hard-code no absolute root (Phase 1 precedent). |

No secrets, no untrusted network input, no SQL, no personal data enter this phase.

## Assumptions Log

| # | Claim | Section | Risk if Wrong |
|---|-------|---------|---------------|
| A1 | The four agens answers (D-03) reach the target's sub-agents as best-effort conversation context, not through a defined input slot. | Q2, Pitfall 2 | If the plan assumes a guaranteed pass-through, the user is re-asked and D-03's intent is missed. Mitigated by labelling them background and setting expectations. |
| A2 | Deploying agens at user level makes `${CLAUDE_SKILL_DIR}/../gsd-ai-integration-phase/SKILL.md` resolve to the installed target. | Q5 | If both a project and a user copy of agens load and the project copy wins, sibling resolution fails; the refuse-on-uncertainty backstop still protects DELEGATE-02. |
| A3 | The `Phase:` integer under `## Current Position` is GSD's authoritative single current-phase signal, and STATE.md never represents two simultaneous active phases. | Q3, Open Questions | If GSD later adds parallel phases, the single-value read would pick one arbitrarily; out of scope this phase. |

## Open Questions

1. **Phase-number selection when multiple GSD phases could be the active phase (deferred question).**
   - What we know: GSD's STATE.md models a single current position — `## Current Position → Phase: N` is one integer, and plans within a phase run sequentially. This project's STATE.md carries exactly one `Phase:` value. The schema does not represent two phases mid-execution at once.
   - What is unclear: whether any real STATE.md state produces genuine ambiguity, and whether agens should ever pass a phase other than the STATE.md current one.
   - Recommendation: read the single `Phase:` integer from `## Current Position` and pass it. Treat an absent or unparseable value as a gate failure and refuse (D-05/D-06) — never enumerate `phases/` directories to guess, which would reintroduce the ambiguity the gate avoids. Do not prompt the user for a phase number this phase (adds an intake step D-08 was designed to skip). Revisit only if GSD introduces parallel phases.

2. **Whether D-03's best-effort context is acceptable, or the plan should reshape the hand-off.**
   - What we know: the target consumes the phase `CONTEXT.md`, not invocation text (Q2).
   - What is unclear: whether the user expects the four answers to fully replace the target's own questions.
   - Recommendation: implement the best-effort background pass and note the limitation in the plan's manual-verification criteria; do not attempt to write into the target's `CONTEXT.md`, which would cross the read/delegate boundary and risk clobbering user decisions.

## Sources

### Primary (HIGH confidence)
- `~/.claude/skills/gsd-ai-integration-phase/SKILL.md` — target frontmatter, `argument-hint`, `allowed-tools`, `$ARGUMENTS` binding, inline execution, orchestration chain. Read in full 2026-07-12.
- `~/.claude/get-shit-done/workflows/ai-integration-phase.md` — the 12-step workflow, `CONTEXT.md` consumption, sub-agent spawns, `AI-SPEC.md` write, AskUserQuestion gates. Read in full 2026-07-12.
- `~/.claude/get-shit-done/templates/state.md` — STATE.md structure, `## Current Position` format, status vocabulary. Read in full 2026-07-12.
- `.planning/phases/01-agens-read-only-recommend/01-PATTERNS.md` and `.claude/skills/agens/SKILL.md` — Phase 1 shipped structure, insertion points, gate-before-showing precedent, vault-root resolution shape. Read in full 2026-07-12.
- `.planning/STATE.md`, `.planning/REQUIREMENTS.md`, `.planning/config.json` — live project state, DELEGATE-01/02, `nyquist_validation: false`. Read in full 2026-07-12.
- `code.claude.com/docs/en/skills` — `Skill` tool invocation, `$ARGUMENTS`/`$N` substitutions, `${CLAUDE_SKILL_DIR}`/`${CLAUDE_PROJECT_DIR}`, `allowed-tools`, install-location table (personal vs project vs plugin), inline vs `context: fork`. Retrieved 2026-07-12.

### Secondary (MEDIUM confidence)
- On-disk enumeration of `~/.claude/skills/` confirming `gsd-ai-integration-phase` present and agens absent at user level; project-level agens confirmed at `.claude/skills/agens/`. Verified via directory listing 2026-07-12.

### Tertiary (LOW confidence)
- None. Every claim is verified against on-disk files or cited from official docs, except the three items in the Assumptions Log.

## Metadata

**Confidence breakdown:**
- Standard stack (Skill tool, substitutions, target contract): HIGH — verified on disk and in official docs.
- Architecture / insertion points: HIGH — read against the Phase 1 SKILL.md line by line.
- Presence-check resolution: MEDIUM — sibling-path approach is sound but depends on the deployment-location decision (A2).
- D-03 pass-through: MEDIUM — the best-effort limitation is verified from the workflow; the user's acceptance is an open question.

**Research date:** 2026-07-12
**Valid until:** 2026-08-11 (stable — Claude Code skill mechanics and GSD conventions move slowly; re-check the Skill-tool substitution table if Claude Code minor version jumps).
