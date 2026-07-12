# Phase 1: agens (Read-Only Recommend) - Context

**Gathered:** 2026-07-12
**Status:** Ready for planning

<domain>
## Phase Boundary

Phase 1 delivers agens' read-only recommend spine: a fixed four-dimension questionnaire, a vault lookup/match against `agent-patterns-index.md`, a citation-resolves verification step, a plain refusal path when nothing fits, dual invocation (explicit slash command and auto-trigger), and the agent-authored tagging convention for anything agens writes. It does not touch framework-fit delegation (Phase 2) or append-only recommendation logging (Phase 4).

</domain>

<decisions>
## Implementation Decisions

### Questionnaire delivery
- **D-01:** All four dimensions (goal, workflow shape, data sensitivity, latency/cost) are asked in a single `AskUserQuestion` turn — one multi-question call — not sequential single-dimension turns and not an infer-then-confirm flow. Keeps the "same four questions every time" guarantee (RECOMMEND-03) visibly literal to the user.
- **D-02:** Every dimension uses fixed multiple-choice options, not free text. Answers must be reproducible and directly matchable against the vault's pattern trigger/trade-off text — RECOMMEND-05's citation-resolves check needs a resolvable match, not prose to interpret.

### Dimension option sets
- **D-03:** Goal is bucketed by capability shape, mirroring the pattern families already in `agent-patterns-index.md` — e.g. "Summarise/transform content", "Answer questions over data", "Automate a multi-step task", "Orchestrate multiple specialists" — so the goal answer maps close to a pattern family instead of needing separate free-text interpretation.
- **D-04:** Workflow shape: "Fixed workflow (predictable steps)" / "Autonomous agent (open-ended, self-directed)" / "Not sure" — mirrors the vault's own `workflow-vs-autonomous-agent.md` concept note directly.
- **D-05:** Data sensitivity: "Public" / "Internal" / "Sensitive or regulated".
- **D-06:** Latency/cost: "Real-time (sub-second)" / "Interactive (seconds)" / "Batch (minutes+, cost-optimised)".

### Claude's Discretion
- Whether agens echoes the four answers back for confirmation before running the vault lookup, or proceeds straight to matching — not decided this session. Default: proceed straight to matching without a separate echo step. The `AskUserQuestion` UI already shows the user's selections before submission; a redundant confirm adds a turn without adding information. Revisit if research surfaces a reason confirmation matters (e.g. users frequently mis-click).
- Exact final wording of the goal-bucket options beyond the four examples in D-03 (including whether a catch-all "something else" bucket is needed) — left to research/planning, guided by the pattern-family language already in `agent-patterns-index.md`.

### Not discussed this session
Three gray areas were presented but the user chose to discuss only "Questionnaire delivery" this session. These are **undecided**, not defaulted — do not assume a trivial answer:
- **Match & refusal threshold** — the concrete rule for what counts as a citation-resolves match (RECOMMEND-05) vs. a "loosely related" citation agens must refuse (RECOMMEND-06).
- **RECOMMEND-07 write scope** — whether Phase 1 writes anything to the vault at all, or whether RECOMMEND-07 only sets the agent-authored tagging convention that Phase 4's logging actually uses.
- **Recommendation output format** — how much of the matched entry gets quoted back, and whether more than one pattern can be surfaced when several genuinely fit.

Research and planning should treat these as open questions to resolve against REQUIREMENTS.md and the `agent-patterns-index.md` entry format, or raise them back to the user if the answer materially changes the plan.

</decisions>

<canonical_refs>
## Canonical References

**Downstream agents MUST read these before planning or implementing.**

### Project-level
- `.planning/PROJECT.md` — Core Value (citation discipline), Constraints, Key Decisions table.
- `.planning/REQUIREMENTS.md` — RECOMMEND-01 through RECOMMEND-07, this phase's full requirement set.
- `.planning/ROADMAP.md` §Phase 1 — goal and success criteria this phase must satisfy.
- `.planning/phases/00-vault-consolidation/00-CONTEXT.md` — Phase 0's decisions (D-01–D-12), especially the index-note structure and the corrected log-format finding (D-10/D-11), and its `<specifics>` note that the user consistently favours matching existing vault convention over inventing new structure.

### Vault — the single lookup target and its precedent
- `wiki-agents/30_Concepts/agent-patterns-index.md` — the single lookup target created in Phase 0. Its bold-pattern-name + trigger + trade-off entry format is what RECOMMEND-05's citation-resolves check matches against, and what D-03's goal-bucket wording should mirror.
- `wiki-agents/30_Concepts/workflow-vs-autonomous-agent.md` — the concept note D-04's workflow-shape option set mirrors directly.
- `wiki-agents/99_Meta/schema.md` — confirms the vault's controlled `topic/*` vocabulary has no entries mapping to "workflow shape", "data sensitivity", or "latency/cost" — these four dimensions are agens' own frame, not a vault taxonomy, so the questionnaire cannot be derived from existing tags.
- `wiki-agents/50_MOCs/MOC - Agent Patterns.md` — the MOC whose dataview query the single lookup target now serves.

### This project's own conventions
- `CLAUDE.md` (repo root) §Technology Stack — the intended `SKILL.md` frontmatter shape for agens (no `disable-model-invocation`/`user-invocable` set on the top-level skill; `allowed-tools: Read Grep Glob AskUserQuestion`), and the directory-name-drives-command-name rule.

</canonical_refs>

<code_context>
## Existing Code Insights

### Reusable Assets
- No Skill code exists yet — `.claude/skills/agens/` has not been created. Nothing to reuse from this repo directly; the reusable asset is documentation, not code: CLAUDE.md's Technology Stack section already specifies the intended frontmatter and tool grants.

### Established Patterns
- `AskUserQuestion` multi-question, single-turn calls are the established mechanism for structured intake elsewhere in this toolchain (this very discuss-phase workflow uses the identical shape) — D-01 reuses that shape rather than inventing a new intake mechanic.

### Integration Points
- The Skill directory name (`agens`) drives the `/agens` slash command, not the `name:` frontmatter field — the two must stay consistent per CLAUDE.md's own "What NOT to Use" table.

</code_context>

<specifics>
## Specific Ideas

Goal-bucket option wording should reuse `agent-patterns-index.md`'s own pattern-family language wherever it fits, rather than inventing new terminology — consistent with the user's Phase 0 pattern of choosing existing vault precedent over a from-scratch design in every gray area discussed (see `00-CONTEXT.md` `<specifics>`).

</specifics>

<deferred>
## Deferred Ideas

None — discussion stayed within phase scope. No scope creep occurred.

</deferred>

---

*Phase: 1-agens (Read-Only Recommend)*
*Context gathered: 2026-07-12*
