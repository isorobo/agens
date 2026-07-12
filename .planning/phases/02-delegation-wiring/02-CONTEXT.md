# Phase 2: Delegation Wiring - Context

**Gathered:** 2026-07-12
**Status:** Ready for planning

<domain>
## Phase Boundary

Phase 2 delivers agens' delegation mechanic: when a user's question is about framework or SDK fit (not agent-pattern fit), agens routes it to `gsd-ai-integration-phase` via the `Skill` tool rather than answering inline, gated on the user being inside an active GSD phase, and fails loudly with a clear message when the delegated skill or the active-phase precondition is missing. It does not touch pattern recommendation (Phase 1, already shipped) or append-only recommendation logging (Phase 4).

**Scouting finding carried into this discussion:** `gsd-framework-selector` — the agent REQUIREMENTS.md/ROADMAP.md name alongside `gsd-ai-integration-phase` — has no Skill-tool entry point. It exists only as an Agent-tool subagent type, spawned internally by `gsd-ai-integration-phase`'s own orchestration (`Select Framework → Research Docs → Research Domain → Design Eval Strategy`). `gsd-ai-integration-phase` is the only one of the two actually reachable via the `Skill` tool. It requires a phase number argument and writes `AI-SPEC.md` into a live GSD phase directory — it is not a lightweight, ad hoc "which framework fits" answerer.

</domain>

<decisions>
## Implementation Decisions

### Delegation target & invocation
- **D-01:** agens always calls `gsd-ai-integration-phase` via the `Skill` tool for framework-fit questions. It never targets `gsd-framework-selector` directly (no Skill entry point exists for it) and never reaches for the `Agent` tool as a workaround to invoke it in isolation.
- **D-02:** Delegation is gated on the user being inside an active GSD phase. Before invoking, agens checks for `.planning/` and a current/in-progress phase (per `STATE.md`). If that precondition isn't met, agens declines — see D-05 (combined pre-flight check).
- **D-03:** agens passes its own four questionnaire answers (goal, workflow shape, data sensitivity, latency/cost) into the `gsd-ai-integration-phase` invocation as background context, so the user isn't asked a re-shaped version of the same questions twice.
- **D-04:** Presence checking is a pre-check, not a reactive catch. agens Globs for the `gsd-ai-integration-phase` skill directory before ever attempting a `Skill` tool call.

### Absence detection & failure message
- **D-05:** The pre-flight check is combined, not two separate checks: it verifies BOTH skill presence (D-04's Glob) AND the active-GSD-phase gate (D-02) before any invocation attempt. One failure message names whichever condition(s) failed, so the user sees every blocking reason at once rather than discovering them one at a time across retries.
- **D-06:** The fixed failure message format is name + reason + next step: state what's missing (skill not installed, or no active GSD phase), why agens is stopping (it routes framework-fit questions rather than answering inline — DELEGATE-01's boundary), and what would fix it (install the GSD skill family / start or resume a GSD phase). This satisfies DELEGATE-02's "explicit, clear failure message" requirement — never a silent inline reimplementation.

### Trigger detection
- **D-07:** agens distinguishes a framework-fit question from its own core pattern-recommendation job by keyword/semantic match on framework and SDK terminology (e.g. LangChain, CrewAI, "Agent SDK", LangGraph, "which framework", "which SDK", "which library should I use") — not a new fifth questionnaire dimension, and not two separately advertised entry points in `SKILL.md`'s "When to Use" section.
- **D-08:** Detection happens upfront, on the user's initial message, before the four-question pattern intake starts. If matched, agens skips the pattern questionnaire entirely and goes straight to the delegation flow (D-05's combined gate check, then D-01's invocation). Framework-fit language surfacing mid-conversation, after a pattern recommendation has already started or been given, is explicitly **not** handled by this phase — see Deferred Ideas.

### Post-delegation handoff
- **D-09:** Once delegation succeeds, agens steps back. `gsd-ai-integration-phase`'s own output (its prompts, its `AI-SPEC.md` artifact, its checkpoints) is presented as-is — not paraphrased, summarized, or re-framed as agens' own recommendation. This keeps the router boundary clean: agens triggers the target Skill and gets out of the way, never absorbing the delegated judgement (ROADMAP.md success criterion 3).

### Claude's Discretion
- Nothing was explicitly deferred to Claude's discretion this session — all four selected gray areas reached explicit user decisions.

</decisions>

<canonical_refs>
## Canonical References

**Downstream agents MUST read these before planning or implementing.**

### Project-level
- `.planning/PROJECT.md` — Core Value (citation discipline), Constraints (delegation discipline: "MCP construction and framework-selection logic must delegate to existing tools... rather than be reimplemented inside agens").
- `.planning/REQUIREMENTS.md` — DELEGATE-01, DELEGATE-02, this phase's full requirement set.
- `.planning/ROADMAP.md` §Phase 2 — goal and 3 success criteria this phase must satisfy.
- `.planning/phases/01-agens-read-only-recommend/01-CONTEXT.md` — Phase 1's decisions, especially the vault-relative-path resolution pattern (D-mirroring approach) and the precedent that this project favours mirroring existing tool/skill conventions over inventing new structure.

### Delegation targets — presence and shape
- `~/.claude/skills/gsd-ai-integration-phase/SKILL.md` — the actual Skill-tool entry point. Frontmatter: `argument-hint: "[phase number]"`, `allowed-tools` includes `Agent`. Orchestrates `gsd-framework-selector → gsd-ai-researcher → gsd-domain-researcher → gsd-eval-planner`. This is what D-01/D-04's Glob pre-check targets and what D-01's `Skill` tool call invokes.
- `~/.claude/get-shit-done/workflows/ai-integration-phase.md` — the workflow `gsd-ai-integration-phase` executes end-to-end; read to understand what agens is actually handing off to (phase-number requirement, `AI-SPEC.md` write target) before planning the invocation shape.
- Agent registry entry for `gsd-framework-selector` — "Presents an interactive decision matrix... Spawned by /gsd:ai-integration-phase and /gsd-select-framework orchestrators." Confirms it is Agent-tool-only, not independently Skill-tool-reachable — the basis for D-01.

### This project's own conventions
- `.claude/skills/agens/SKILL.md` — existing Phase 1 skill body (four-question intake, `AskUserQuestion` shape, vault-relative path resolution pattern) that D-03's questionnaire-context handoff and D-07/D-08's upfront-detection logic must integrate with, not duplicate.
- `.claude/skills/agens/references/agent-authored-convention.md` — Phase 1's tagging convention; check whether an agent-authored tag applies to anything Phase 2 might touch (none currently expected, since Phase 2 delegates rather than writes).

</canonical_refs>

<code_context>
## Existing Code Insights

### Reusable Assets
- `.claude/skills/agens/SKILL.md` Step 1 (the four fixed questions, single `AskUserQuestion` call) — D-03 reuses these exact answers as delegation context; no new intake mechanic needed.
- `.claude/skills/agens/SKILL.md`'s vault-root resolution pattern (resolve via granted additional directories, refuse plainly if unresolvable) — the same "resolve first, refuse plainly on failure" shape applies to D-04/D-05's skill-presence and phase-gate pre-check.

### Established Patterns
- Read+Grep/Glob gate-before-showing-anything is already agens' house style from Phase 1's citation gate (Step 2b) — D-04/D-05's pre-flight check before invocation is the same pattern applied to skill presence and phase-gate, rather than a new failure-handling idiom.
- Plain, fixed refusal text (Phase 1's Step 3 refusal block) is the precedent for D-06's fixed failure message — name what was searched, state nothing was found, offer no partial/decorative substitute.

### Integration Points
- New logic sits in `.claude/skills/agens/SKILL.md` as new steps: an upfront trigger-detection step (D-07/D-08) ahead of the existing Step 1 four-question intake, and a delegation flow (D-01/D-02/D-04/D-05/D-06/D-09) that runs instead of Steps 2-3 when triggered.
- `gsd-ai-integration-phase` is invoked via the `Skill` tool — the same cross-skill mechanism CLAUDE.md's Technology Stack section already names as "the sanctioned cross-skill mechanism" for agens' MCP Builder handoff, now confirmed as the mechanism for this handoff too.

</code_context>

<specifics>
## Specific Ideas

None beyond the decisions captured above — the discussion stayed at the mechanism level (what to call, when, how to detect, how to fail) rather than surfacing example phrasings or reference UX the user had in mind.

</specifics>

<deferred>
## Deferred Ideas

- **Mid-conversation trigger detection** — a framework-fit question surfacing after the pattern questionnaire has already started, or after a pattern recommendation has already been given, is out of scope for this phase (D-08 restricts detection to the user's initial message). Revisit if this proves too limiting in practice.
- **Phase-number selection when multiple GSD phases are candidates** — D-02's active-phase gate assumes `STATE.md`'s current/in-progress phase is the right one to pass to `gsd-ai-integration-phase`; the user was not asked whether agens should ever prompt for a different phase number, and this was not settled either way this session. Research/planning should treat this as an open question if `STATE.md`'s current phase can be ambiguous (e.g. multiple phases mid-execution).

</deferred>

---

*Phase: 2-Delegation Wiring*
*Context gathered: 2026-07-12*
